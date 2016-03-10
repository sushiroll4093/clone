# Ruby Coding Standards

We generally follow the [Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide) maintained by bbatsov, with the following exceptions:

## Coercing Boolean Values
It is acceptable to use `!!` to coerce a value to be `true` or `false`.

## Collections
It is acceptable to have a comma after the last item of an `Array` or `Hash` literal.

## Exceptions
`raise` is acceptable to use instead of `fail`.

## Lambdas
Stabby lambdas (`->`) are preferred for both single line and multi-line blocks. They solve a syntactic ambiguity with pipes being used in defaults for arguments.

## Line Length
The longest line in the Ruby ./app/ codebase is 350-characters long. Never write a line of code that is longer than this.

The median longest line of code in any given Ruby file is 118 characters. Avoid writing a line that is longer than this.

About 93% of the Ruby codebase is composed of lines of fewer than 80 characters. Aspire to write all your code in this 93%. 96.5% of our Ruby lines are 100-characters or less in length. It is acceptable to write code that is 100-characters or less long.

## Perl-ish Variables
Ruby allows several [special variables](http://ruby.wikia.com/wiki/Special_variable) which it primarily inherited from Perl. These are a $ followed by one other character. Avoid their use, except for the following, as they are better known than any alternative:
- `$?` which returns Process::Status for last executed child process (e.g., from ls -l).
- `$!` which returns the pending exception. This is the second parameter to a raise statement.

Use named captures and [Regexp#match](http://ruby-doc.org/core-2.2.0/Regexp.html#method-i-match) rather than Perl variables (`$~`, `$1`). See [Refactoring Regular Expressions with Ruby 1.9 Named Captures](https://www.bignerdranch.com/blog/refactoring-regular-expressions-with-ruby-1-9-named-captures/).

Note that =~ is allowed when you don't have any captures. Regexp.last_match is not allowed.

## Post-loop conditionals
While a loop … break construction is preferred, post-loop conditionals are acceptable.

## While/Until
While we prefer single-line while/until (https://github.com/bbatsov/ruby-style-guide#while-as-a-modifier), use common sense if the line length gets too long. Break it into a multi-line loop.

## Long Line Method Invocation
Prefer normal indent over indenting to the first paren:
```ruby
# bad
def this_is_especially_bad_with_long_method_names(first_attribute: variable_foo,
                                                  second_attribute: variable_bar,
                                                  third_attribute: variable_baz)
end

# good (normal indent)
def send_mail(source)
  Mailer.deliver(
    to: 'bob@example.com',
    from: 'us@example.com',
    subject: 'Important message',
    body: source.text
  )
end
```
No one likes to wrangle whitespace. Ref: https://github.com/bbatsov/ruby-style-guide#no-double-indent
