# ActiveRecord & SQL Query Style Guide

## Condition Syntax

`update_all` and `where` both use "condition" syntax. Condition syntax is either a literal string of SQL, an array with the first element being SQL and insertion markers, and the rest being the insertions, or a hash. The following rules apply:

* Hash syntax is preferred (if possible), then array, then literal:

  ```ruby
    where(user_id: user)        # FIRST CHOICE   :)
    where("user_id=?", user)    # SECOND CHOICE  :|
    where("user_id=#{user.id}") # THIRD CHOICE   :(  This is a good way to allow SQL injection attacks.
  ```

  Remember, this applies to `update_all`:

  ```ruby
    update_all(user_id: user)
  ```

  Also to literals (scalar or arrays):

  ```ruby
    where(workflow_state: 'deleted')
    where(workflow_state: ['invited', 'active'])
  ```

  And even to betweens:

  ```ruby
    # BETTER
    where(id: min..max)
    # WORSE
    where("id>=? AND id<=?", min, max)
  ```

  * Hash syntax is preferred for several reasons. It is easier (read: possible) for sharding to parse to do automatic shard inference and id translation for you, is less susceptible (not possible) to SQL injection, allows duplicate-condition folding, and allows for proper per-database quoting of column names (such as the from column on the messages table).

  * Nested hash syntax is preferred over string keys:

    ```ruby
      # BETTER
      where(enrollments: { user_id: user, workflow_state: 'active')
      # WORSE
      where('enrollments.user_id' => user, 'enrollments.workflow_state' => 'active')
    ```

* If hash syntax is not possible, don't use insertions for string and integer literals:

  ```ruby
    # BETTER
    where("workflow_state<>'deleted'")
    # WORSE
    where("workflow_state<>?", 'deleted')
  ```

  Once we've left hash syntax, we lose its benefits, so don't make it do extra work of doing an insertion. Insertions are still necessary for boolean and date literals, since those are formatted differently depending on the database.

* Don't use extraneous braces and brackets. `where` (but *not* `update_all`) takes a splat, so no need to use brackets there:

  ```ruby
    # BETTER
    where(user_id: user)
    # WORSE
    where({user_id: user})

    # BETTER
    where("created_at<?", 1.week.ago)
    # WORSE
    where(["created_at<?", 1.week.ago])

    # WORKING
    update_all(["updated_at=created_at, user_id=?", new_user])
    # NOT WORKING!
    update_all("updated_at=created_at, user_id=?", new_user)
  ```

* Pass AR objects instead of IDs if you have them. This reduces code (no `.id` or `map(&:id)` or `user.is_a?(User) ? user.id : user`), and is also helpful for sharding because id translation is delayed until the shard the query will actually run on is active:

  ```ruby
    # BETTER
    where(user_id: user)
    # WORSE
    where(user_id: user.id)
    where(user_id: user.try(:id))
    where(user_id: (user.is_a?(User) ? user.id : user))
    where(user_id: users.map(&:id)
  ```

  This also applies to `update_all`, and array syntax (`update_all("created_at=updated_at, user_id=?", user)`). This even means you don't have to worry about writing a different `IS NULL` query (if using hash syntax).

* When using SQL, don't have whitespace between non-word operators. This slightly reduces network traffic, but more importantly lets ops and monitoring see more of a query in a very small space:

  ```ruby
    # BETTER
    where("workflow_state<>'deleted'")
    # WORSE
    where("workflow_state <> 'deleted'")
  ```

* When using SQL, use double quotes. Thus your string literals will work with single quotes:

  ```ruby
    # BETTER
    where("workflow_state<>'deleted'")
    # WORSE
    where('workflow_state<>?', 'deleted')
  ```

  (notice that single quotes forces me to either use an insertion when I shouldn't or escape the single quote itself.)

* Prefer `<>` over `!=` in SQL

  ```ruby
    # BETTER
    where("workflow_state<>'deleted'")
    # WORSE
    where("workflow_state!='deleted'")
  ```

## `select`, `order`, and `group`

`select`, `order`, and `group` can take an array and symbols now. Rails will uniq duplicate columns.

 * Prefer array and symbols if possible:

  ```ruby
    # BETTER
    select([:id, :name])
    # WORSE
    select("id, name")
  ```

 * `order` and `group` can take a splat, so omit the brackets:

  ```ruby
    # BETTER
    order(:id, :name)
    # WORSE
    order([:id, :name])
  ```
* use hash for desc `order` rather than raw SQL:
```
# BETER
order(id: :desc)
# WORSE
order("id DESC")
```

## `distinct`

`distinct` is a thing. It's preferred to use `distinct` and make a scope, rather than use a `select("DISTINCT something")`:

  ```ruby
    # BETTER
    select(:id).distinct
    # WORSE
    select("DISTINCT id")
  ```

## `pluck`

`pluck` is a thing. It saves instantiating AR objects (not a cheap process).

  ```ruby
    # BETTER
    pluck(:id)
    # WORSE
    select(:id).map(&:id)
  ```

## `first` and `last` (and `order`)

`first` and `last` will automatically add an order (by primary key, or reverse an existing order) if necessary.

  ```ruby
    # BEST
    User.last
    User.order(:position).last
    # BETTER
    User.order(:id).last
    # WORSE
    User.order(id: :desc).first

    # Rails screws this up ("ORDER BY COALESCE(id DESC, 0) DESC")
    User.order("COALESCE(id, 0)").last
    # so you have to do "WORSE"
    User.order("COALESCE(id, 0) DESC").first
  ```

## `exists?`

`exists?` can be used to check if there are any records matching the query. Typically in the past we've written this as `query.first.present?`, `!!query.first`, `query.empty?` or `query.count > 0`, simply due to ignorance of `exists?`. `exists?` is the preferred method. `count > 0` is bad because it has to count all rows, which can be much slower than just checking if at least one row exists. Be kind to your DB.

  ```ruby
    # BETTER
    User.where(id: ids, workflow_state: 'active').exists?
    # WORSE
    User.where(id: ids, workflow_state: 'active').first.present?
    !!User.where(id: ids, workflow_state: 'active').first
    # EVEN WORSE
    User.where(id: ids, workflow_state: 'active').count > 0
    User.where(id: ids, workflow_state: 'active').any?
  ```

## default_scope

Just don't. See Associations.

## Associations

Do _not_ put options on your associations. This just causes a lot of pain later. Remember, it's easier to _add_ to a scope than to remove from it.

  ```ruby
     # BETTER
     has_many :assignments
     def self.assignments_with_submissions
       assignments.include(:submissions)
     end

     # WORSE
     has_many :assignments, -> { includes(:submissions) }

     course.assignments.where(assignment_group_id: 1).each {} # just loaded a TON of submission objects, that you probably didn't need

     # BETTER
     # course.rb
     has_many :enrollments
     # enrollment.rb
     scope :active, -> { where(workflow_state: 'active') }

     # WORSE
     # course.rb
     has_many :enrollments, -> { where(workflow_state: 'active') }

     # in a datafixup
     course.enrollments.where(user_id: u).delete_all # we miss data, assuming enrollments means, you know, enrollments, regardless of state
  ```

## Preloading

Preloading can be a huge performance boon, by avoiding N+1 queries. However, you must do it carefully.

Prefer `eager_load`/`preload` over `includes`. `eager_load` always does a `LEFT OUTER JOIN`; `preload` always splits it into two queries. `includes` may do either, _even depending on the contents of user provided data_. This can be especially bad if the preloaded data crosses shard boundaries - you just broke your query! Usually you want to do `preload` - doing a `LEFT OUTER JOIN` makes very hard to understand and reason-about queries, and actually transfers a lot more data because the non-changing data is duplicated with each row in a one-to-many.

  ```ruby
  # GOOD
  course.enrollments.preload(user: :communication_channels).where(type: some_variable)

  # BAD
  # if some_variable happens to contain the string "users." or "communication_channels.", then Rails will use
  # a left outer join in a single query. Not only is the SQL very hard to read, but it also breaks in a
  # multi-shard environment. It will assume it got all the data, even though some users may belong on a
  # different shard, so it won't pull in their communication_channels from that shard.
  course.enrollments.includes(user: :communication_channels).where(type: some_variable)

  # GOOD
  course.enrollments.eager_load(:user).where(users: { name: 'Cody' })
  ```

## Relation vs. Array

Sometimes you want a relation (to continue chaining query methods). Sometimes you want an array (to further manipulate it, or just to access it a lot). In Rails 2 and Rails 3, `#all` will give you an array. In Rails 4, `#all` just returns another relation. Use `#to_a` instead; it works in all versions of Rails.

## Querying the slave

Some queries you just can't optimize. You'll want to run them on the slave:

  ```ruby
  Shackles.activate(:slave) do
    some_really_expensive_query
  end
  ```

## `none`

Sometimes you're building up a relation, and then discover that it shouldn't return any results. However, you can't return an empty array, because the caller expects a relation. Instead of using a dummy condition that can never be true, use the `none` scope. It behaves like a relation, but will avoid even going to the database at all.

  ```ruby
  def students_that_basket_weave
    students = self.students
    unless feature_enabled?(:basket_weaving)
      return students.none
    end
    students
  end
  ```

## Subqueries

ActiveRecord can take a subquery as a condition. This can be useful in various ways. For example, a `NOT EXISTS` query _can_ be more performant than doing a join, or for forming a query slightly different than an association would.

  ```ruby
  User.where(id: list_of_users).where("NOT EXISTS (?)", Enrollment.active.where("user_id=users.id")).to_a
  # Returns users that don't have any active enrollments (as opposed to checking them one at a time with
  # list_of_users.select { |u| !u.enrollments.exists? } )
  # => SELECT "users".* FROM "users"  WHERE "users"."id" IN (1, 2) AND (NOT EXISTS (SELECT "enrollments".* FROM "enrollments"  WHERE (enrollments.workflow_state<>'deleted') AND (user_id=users.id)))
  ```

## `merge`

You can merge associations to easily find a many-to-many record:

  ```ruby
  assignment.submissions.merge(user.submissions.scoped).first
  # Returns the user's assignment submission, without having to explicitly add the conditions from both associations
  # => SELECT "submissions".* FROM "submissions"  WHERE "submissions"."assignment_id" = 1 AND "submissions"."user_id" = 6
  ```

## Raw SQL

When using raw SQL, _always_ quote the table name. This includes in joins. This allows us to use multiple shards in the same database, without switching users or schema search paths:

  ```ruby
  # GOOD
  User.connection.update("UPDATE #{User.quoted_table_name} SET #{some_really_complicated_dynamic_set}")
  Assignment.joins("INNER JOIN #{Course.quoted_table_name} ON context_id=courses.id")

  # BAD
  User.connection.update("UPDATE users SET #{some_really_complicated_dynamic_set}")
  Assignment.joins("INNER JOIN courses ON context_id=courses.id")
  ```

# Canvas Extensions

## Joins on Update and Delete

Vanilla Rails doesn't support joins on updates or deletes. Canvas does, by using Postgres and MySQL specific syntax for it.

  ```ruby
  Enrollment.joins(:course).where(courses: { root_account_id: 1 }).update_all(updated_at: Time.now.utc)
  # => UPDATE "enrollments" SET "updated_at" = '2015-01-27 17:12:04.127305' FROM "courses" WHERE "courses"."root_account_id" = 1 AND ("courses"."id" = "enrollments"."course_id")
  ```

## Offset and Limit on Update and Delete

In data fixups, you may need to update or delete a lot of rows. If you do a normal update or delete, it will do the entire table at once. You could use a find_ids_in_batches or find_ids_in_ranges, but that may force a non-optimal query plan (essentially a single pass over the entire table, even though there's a useful index on one of your conditions). Instead, just loop around a limited update_all, and it will use the index, but only do a batch at a time. Notice that it simply does a subquery. This can also be useful to delete all but the first record from a scope (i.e. when deleting duplicates):

  ```ruby
  while course.enrollments.limit(1000).update_all(updated_at: Time.now.utc) > 0; end
  # => UPDATE "enrollments" SET "updated_at" = '2015-01-27 17:15:04.313610' WHERE "enrollments"."id" IN (SELECT "enrollments"."id" FROM "enrollments" WHERE "enrollments"."course_id" = 1 AND (enrollments.workflow_state != 'deleted') LIMIT 1000)
  duplicate_scope = Enrollment.where(user_id: 1, type: 'StudentEnrollment', course_id: 2)
  duplicate_scope.offset(1).delete_all
  ```

## `bulk_insert`

If you have a lot of data to insert, and don't need to run ActiveRecord callbacks and such, you can use `bulk_insert`. It's even faster than running a transaction with a bunch of INSERT statements in it. It just takes an array of hashes:

  ```ruby
  def bulk_insert_participants(user_ids, options = {})
    options = {
        :conversation_id => self.id,
        :workflow_state => 'read',
        :has_attachments => has_attachments?,
        :has_media_objects => has_media_objects?,
        :root_account_ids => self.root_account_ids
    }.merge(options)
    ConversationParticipant.bulk_insert(user_ids.map{ |user_id|
      options.merge({:user_id => user_id})
    })
  end
  ```
