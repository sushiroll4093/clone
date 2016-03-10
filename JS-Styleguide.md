# JS Coding Standards

## AirBnB
Use [AirBnB's styleguide](https://github.com/airbnb/javascript) with the exception
to items listed below.

### Exceptions where AirBnB is not followed

#### Naming Conventions

When you are dealing with things that were .to_json'ed from ruby, they will be in ruby's `underscored_method_name` syntax


## Escaping
To mitigate XSS, here are a few guidelines:

- never ever ever manually construct html snippets (e.g. `$foo.html("<b>" + zomgxss + "</b>")`. instead, prefer `$.fn.text` or handlebars templates to interpolate user data into some html.
- along those lines, never ever ever use `$.fn.html` to set content (unless it's a handlebars template). even if you're sure it's safe, a later refactor/change could easily make it unsafe. so don't do it.
- if you need to intersperse HTML into your I18n (say, a link), you should really use a handlebars template. to be fair, the i18nwrapper functionality auto-escapes interpolated data so that the result is i18n-safe (e.g. `I18n.t("foo", "ohai *%{name}*", {name: "<script>alert('lol)</script>", wrapper: "<b>$1</b>"})` becomes `"ohai <b>&lt;script&gt;alert('lol)&lt;/script&gt;</b>"`). that said, I18n.t doesn't normally html-escape the result (or interpolated data). because of this, a safe string could easily become unsafe or double-escaped in a refactor of the content. so just use handlebars. it's the same end result, but you don't risk introducing XSS when you refactor.
- there is an htmlEscape helper, but you should be writing/refactoring code in such a way that we never need to explicitly use it.

## Rails
### How do I get variables from rails into my js?

There should never be any case to ever do `<% js_block do %>` or have hidden elements with important data floating around on the page, instead from rails use `js_env`.

example:

```ruby
# from a controller or view
js_env :TOPIC => {
  :ID => @topic.id,
  :URL => named_context_url(@context, :context_discussion_topic_url, @topic),
  :PERMISSIONS => {
    :CAN_REPLY => !(@topic.for_group_assignment? || @topic.locked?)
  }
}

# in your coffeeScript
$.ajax ENV.TOPIC.URL
```

## jQuery

### Don't `return false` inside of jQuery event callbacks
Be explicit about what you are wanting to do. Usually, you just want to `e.preventDefault()` if you are handling a click on something (so it doesn't follow the link).  If you do `e.stopPropigation()`, put an inline comment of why, and only do it if you really have to.

another pattern is to do
```js
define(['compiled/fn/preventDefault'], function(preventDefault) {
  return $('foo').click(preventDefault(function() {
    return console.log("this fn doesn't care about the event arg");
  }));
});
```
