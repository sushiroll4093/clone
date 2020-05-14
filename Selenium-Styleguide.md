## Guidelines for writing good selenium specs

### What should NOT be tested with selenium

- animations

- Redundant page loads (e.g. Only test a link or button that goes to another page if its been broken before. In a helper, avoid loading a page then clicking another link to get to a page. In a before_filter, don't load a page, modify data, and then refresh the page.)

- Anything else multiple times, for example, test creation of items with selenium once, but if that data is needed in another spec, just use database seeding.

### Standards and Conventions

**Commit Message:** any commit dealing with just specs needs to have the commit message prefixed with spec: _example_ - "spec: adding selenium specs to assignments spec"

## Selector best practices

- don't depend on element types, or exact descendants from parents. Use .some_selector not div.some_selector. Explicitly looking for a div, p, span, a, or an h4 is bad because if we have to change the html to restyle or something then the test breaks. Even adding the div in div.some_selector is superfluous. One exception: elements that that you can't swap out a different element for like a select or textarea(select.course_id is fine). We try to enforce the same policy on the javascript we write (ie: \$('#something div p').click... is discouraged) and anything submitted to the server via normal html forms depends on having names and/or ids) so there should always be some combination of id and/or classes you can use to select an important element.

- don't use xpath. especially stuff like '//div[@class="something_special"]/div[3]/span[2]/select'. That gives no indication of what you are selecting and will break if any html structure is changed/added. besides we are all use to using css selectors for stylesheets and jQuery.

- be careful to reasonably ensure that you are selecting the right thing on the page. ex: `f('.name')` could find the wrong link if there was anything else with the class .name (highly likely) on the page. Doing something like #user_details_form .name would be better.

- to find elements use the helper methods `f('some selector')`, `ff('some selector')`, `fj('some selector')`, `ffj('some selector')`
  _ f - use instead of `driver.find_element` - returns first matched element
  _ ff - use instead of `driver.find_elements` - returns all matched elements
  _ fj - use instead of `find_with_jquery` - returns first matched element, only use if you are doing jquery specific things - example - `fj('some_selector:nth-child(3)')`
  _ ffj- use instead of `find_all_with_jquery` - returns all matched elements, only use if you are doing jquery specific things - `ffj('some_selector:visible')`
  <br />
  <br />
- dont rely on the english version of what a link text says. ex: dont do driver.find_element(:link, 'New Topic') if we ever run the specs testing out the ui in a different language that will break.

- just so you know, I try (and encourage others) to stick to a scheme where classes and ids that are just for css styling use dashes but classes/ids that are there for javascript hooks use underscores so .no-text-shadow would be a less useful selenium selector (purely cosmetic) than #thing_that_tiggers_dialog (probably tied to js)

- when you find a key token (like seeing a `<select name="assignment_to_never_drop">` ) it is often useful to do a global search for "assignment_to_never_drop" in the project and look for javascript that uses that token and use that as a clue for what the selector that the javascript is using to add it's functionality (in this case you would find `$("select[name='assignment_to_never_drop']")` in a js file).

- you can assume that if the javascript uses that selector then you can use it and it is not going to be brittle because if something changes, the javascript will break too.
- The order of things you should look for when selecting is -

- The name attribute if it is an input/textarea/select (because changing that name would mean we changed not just client side code but also server side code, in fact it would probably mean we changed a database column name: `<input name="discussion\_topic[title] />` maps directly to the title column in the discussion_topics table, that is probably not going to change).
- `#an_id_that_is_underscored`: Since you can assume an id is unique to the page, you wont accidentally select something you didn't mean to and because it uses underscores you can assume the js uses it too.
- `#an_id` .followed_by_an_underscored_class: again, by using an id you know you are starting from the right spot and using an underscored class name means the js probably uses it too.

## Selector examples: (bad example _=>_ good example)

- `h2.title` **=>** `#content .title`

- `//div[@class="form_rules"]/div[2]/select` **=>** `#add_group select.rule_type`

- `//div[@class="form_rules"]/div[3]/span[@class="never_drop_assignment"]/select` **=>** `select[name='assignment_to_never_drop']`

- `//div[@class="form_rules"]/div[2]//input` **=>** `.form_rules [name="scores_to_drop"]`

- `div.something` **=>** `.something`

- `#edit_dialog form div input.title` **=>** `#edit_dialog input.title ('form' and 'div' dont add any value)`

- `.anything_else_before div#an_id.or-along-with-it` **=>** `#an_id` (since it is an id you can assume it is unique to the page)

- `f('button[type="submit"]').click` **=>** `f(‘form_selector’).submit`

- `f(‘#some_id’).find_element(:css, ‘.some_css’)` **=>** `f(‘#some_id .some_css’)` (chaining finds results in two separate requests from the client, so it slows down the test)

## PostgreSQL selector example: (bad example => good example)

- `f(“#context_module_item_2)` - (don’t expect the tag ID to always be the same) **=>** `tag = ContentTag.last f("#context_module_item_#{tag.id}")`

## ID, CSS, and other selenium type examples (most common uses)

- `f(‘.some_class’)` - use this one for class attribute or when chaining selectors together
- `f(‘#some_id .some_class’)` - chaining example

## Helper methods that should be used

clicking a select option -

- `click_option(‘css_selector’, ‘option_text’)` - there is an option third parameter that you can pass to override the default text option.
- override example - `click_option(‘css_selector’, ‘some_value’, :value)`

wait_for_animations - add this to wait for jQuery animations

wait_for_ajax_requests

wait_for_ajaximations - add this if ajax requests and animations are happening

expect_new_page_load { button_element.click - example } - use when you are doing any action that triggers a page load.

wait_for_js - add this if you have having errors after a page load, with AMD selenium might get in the thread and start doing its thing before the page is ready

keep_trying_until { element.should be_displayed - example }

assert_flash_notice_message /regex/

assert_flash_error_message /regex/

is_checked(‘css_selector’)

submit_form('form_element') or submit_form('form_css')

submit_dialog('dialog_element') or submit_dialog('dialog_css')

replace_content(element, ‘text’) - use this always, less lines of code, better readability
this is the wrong way to do it
f("some_selector").send_keys(:backspace)
f("the_same_selector").send_keys(‘some text’)

set_value(input, value) - use this if you need the onChange event to be fired. This works the same as replace_content

close_visible_dialog

first_selected_option(select_element) - returns the first option element in the select element

make_full_screen

resize_screen_to_default - use this after you are done with make_full_screen

validate_link(link_element, ‘breadcrumb_text’)

element_exists - element_exists("some_selector").should be_false

have_class - f(‘some_selector’).should have_class(‘class_text’)

have_attribute - f('some_selector').should have_attribute('data-id', 'expected value for data id')

include_text - f(‘some_selector’).should include_text(‘some_text’)

drag_with_js(‘selector’, x, y) - use this instead of driver.action.drag_and_drop, x and y represent how many pixels you want to drag and drop the matched element

## Context / Describe

Context: use context when you are testing the same feature but in a different way

- example - context “calendar as a teacher”, context “calendar as a student”

Describe: use describe when you are testing a feature or certain part of a feature

- example - describe “main calendar”, describe “mini calendar”, describe “calendar context list”

## Selenium Spec File Standards (Common Example)

```ruby
require File.expand_path(File.dirname(__FILE__) + '/common') #helper methods

describe "name of the feature you are testing" do
  it_should_behave_like "in-process server selenium tests"

  before (:each) do
    #put anything code here that should run before each contained within this
    describe
  end

  def method_for_both_contexts
    #this method is scoped within the main feature describe and can be used in
    any spec as long as the spec is scoped within the main feature describe as
    well
  end

  context "example as a teacher" do
    def method_for_teacher_context
      #this method is scoped for the teacher context and can only be used by
      specs in the teacher context
    end

    before (:each) do
      #put any code here that all specs in the context are going to need
      before each run
      course_with_teacher_logged_in
    end

    it "should be a good description for the test case" do
      #spec code
    end
  end

  context "example as a student" do
    it "should be a good description for the test case" do
      def spec_method
        #this method is only scoped for the current spec
      end

      #spec code
    end
  end
```

# Other random selenium advice

- As with all spec writing, use factories as frequently as possible, that way if a model changes, fewer specs need to be fixed.
- marking specs as pending -

```ruby
it "should be a good description for the test case of the feature" do
  #marking a spec as pending with no do end block doesn't run the spec
  pending("reason why spec is marked pending")
end

it "should be a good description for the test case of the feature" do
  #marking the spec as pending with a do end block
  #this means that all the code in the block will be run
  #if the code returns an error, it will be marked as pending
  #if the code passes all assertions, the spec will fail
  #the failure message will tell you to remove the pending because the spec passes
  pending("reason why spec is marked pending") do
    #spec code
  end
end
```

- try not to use wait_for_dom_ready or keep_trying_until. they really slow down the tests; keep_trying_until contains a sleep. if you are thinking of throwing one in, you may be doing something wrong. But yes, sometimes they are necessary.

- if you are getting element not found in cache error make sure the page hasn’t done a refresh or there has been an ajax reload on the element you are checking for. This mainly happens when you save an element into a variable and the page reloads so there isn’t a reference to that element anymore. If you run into any other element caching issue, try using find_with_jquery, since it bypasses the selenium element cache.

- if you are ever confused about when to use context / describe / shared examples take a look in all of the files first and don’t be afraid to ask.

- remember, you are validating the UI - example: it is good to make sure when adding an assignment that the assignment count in the database goes up by 1. You also need to check that it is displayed in the UI as well.

- if you add a user stick to the naming convention - anything@example.com

- uploading files - most basic syntax example -

```ruby
name, path, data = get_file({:text => 'testfile1.txt', :image => 'graded.png'})
f('.file_name').send_keys(path)
f('.upload_button').click
```
