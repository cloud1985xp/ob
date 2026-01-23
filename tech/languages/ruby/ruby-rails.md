---
tags:
  - ruby
  - rails
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Ruby & Rails

Created: 2021年5月24日 下午11:59

[Best of Ruby Gems Series](https://planetruby.github.io/gems/)

[Home](https://blog.arkency.com/)

# Ruby Programming

### Yield, Yield_Self

[Understanding Yield & Yield_Self in Ruby (Complete Guide)](https://www.rubyguides.com/2019/12/yield-keyword/)

### RubyConf 2019 @China

[ruby-china/RubyConfChina2019Slides](https://github.com/ruby-china/RubyConfChina2019Slides)

### Code Snipet

### Encode PNG with Base64

[https://gist.github.com/hateradio/3c2cae39d432dfbe4647](https://gist.github.com/hateradio/3c2cae39d432dfbe4647)

# Rails Development

[Rails 啟動過程 - Ruby on Rails 指南](https://rails.ruby.tw/initialization.html)

## JSON API

### serializer / jbuilder alternative

[jsonapi-rb | Efficiently build and consume JSON API documents.](http://jsonapi-rb.org/guides/getting_started/rails.html)

[GitHub - jsonapi-serializer/jsonapi-serializer: A fast JSON:API serializer for Ruby (fork of Netflix/fast_jsonapi)](https://github.com/jsonapi-serializer/jsonapi-serializer)

## Hotwire

[HTML Over The Wire | Hotwire](https://hotwire.dev/)

## Arel

[Arel cheatsheet](https://devhints.io/arel)

## Security of Rails

[Common Rails Security Pitfalls and Their Solutions - SitePoint](https://www.sitepoint.com/common-rails-security-pitfalls-and-their-solutions/)

### Payment

pay2go

[https://github.com/imgarylai/active_merchant_pay2go/blob/master/lib/offsite_payments/integrations/pay2go/notification.rb](https://github.com/imgarylai/active_merchant_pay2go/blob/master/lib/offsite_payments/integrations/pay2go/notification.rb)

tappay

[https://docs.tappaysdk.com/tutorial/zh/portal.html#merchant-notification](https://docs.tappaysdk.com/tutorial/zh/portal.html#merchant-notification)

[https://github.com/hzchirs/tappay-ruby](https://github.com/hzchirs/tappay-ruby)

[https://yu-jack.github.io/2017/09/23/tappay-payment/](https://yu-jack.github.io/2017/09/23/tappay-payment/)

### SSO

[https://github.com/joshsoftware/sso-devise-omniauth-provider](https://github.com/joshsoftware/sso-devise-omniauth-provider)

[https://github.com/joshsoftware/sso-devise-omniauth-client](https://github.com/joshsoftware/sso-devise-omniauth-client)

### ERB Template Rendering

[http://brianwhitmer.blogspot.com/2009/07/ruby-erb-and-outputbuffer.html](http://brianwhitmer.blogspot.com/2009/07/ruby-erb-and-outputbuffer.html)

[https://hostiledeveloper.com/2015/05/28/working-with-templates-in-ruby-erb.html](https://hostiledeveloper.com/2015/05/28/working-with-templates-in-ruby-erb.html)

[https://www.bridgetownrb.com/docs/erb-and-beyond](https://www.bridgetownrb.com/docs/erb-and-beyond)

[https://medium.com/rubyinside/disassembling-rails-template-rendering-1-51795f579577](https://medium.com/rubyinside/disassembling-rails-template-rendering-1-51795f579577)

[https://www.clever-cloud.com/blog/engineering/2019/03/28/understanding-rails-templating-part-1/](https://www.clever-cloud.com/blog/engineering/2019/03/28/understanding-rails-templating-part-1/)

## Feature Flag

[jnunemaker/flipper](https://github.com/jnunemaker/flipper)

[mgsnova/feature](https://github.com/mgsnova/feature)

### Memory / Monitoring Process Memory

[The Limits of Copy-on-write: How Ruby Allocates Memory](https://brandur.org/ruby-memory)

[schneems/get_process_mem](https://github.com/schneems/get_process_mem)

## Autoload Classic Mode vs Zeitwerk Mode

[Autoloading and Reloading Constants (Zeitwerk Mode) - Ruby on Rails Guides](https://guides.rubyonrails.org/autoloading_and_reloading_constants.html)

[Autoloading and Reloading Constants (Classic Mode) - Ruby on Rails Guides](https://guides.rubyonrails.org/autoloading_and_reloading_constants_classic_mode.html)

### DB Sharding in Rails

[Add support to `connected_to` and `connects_to` for horizontal sharding by seejohnrun · Pull Request #38531 · rails/rails](https://github.com/rails/rails/pull/38531)

Implemented by Sekiyama san

[aktsk/octoball](https://github.com/aktsk/octoball)

## Rails 6

[Rails 6.1: Horizontal Sharding, Multi-DB Improvements, Strict Loading, Destroy Associations in Background, Error Objects, and more!](https://weblog.rubyonrails.org/2020/12/9/Rails-6-1-0-release/)

### Hash#deep_transform_values

[Rails 6 adds Hash#deep_transform_values and Hash#deep_transform_values!](https://blog.saeloun.com/2019/09/10/rails-6-adds-hash-deep_transform_values.html)

### ActionText

[Action Text Overview - Ruby on Rails Guides](https://edgeguides.rubyonrails.org/action_text_overview.html)

[Handling attachments in Action Text in Rails 6](https://blog.saeloun.com/2019/11/12/attachments-in-action-text-rails-6.html)

[Using ActionText in Rails 6 | OOZOU](https://oozou.com/blog/using-actiontext-in-rails-6-85)

ActionText uses 'trix'

[basecamp/trix](https://github.com/basecamp/trix)

[WYSIWYG Editing and Drag-and-Drop Image Upload with Trix and Shrine](https://www.headway.io/blog/how-to-use-trix-and-shrine-for-wysiwyg-editing-with-drag-and-drop-image-uploading)

Adjust min-height of trix editor

[Trix WYSIWYG Editor change default rows/vertical height of textfield](https://stackoverflow.com/questions/50768504/trix-wysiwyg-editor-change-default-rows-vertical-height-of-textfield)

### Other WYSWYG Solution

[The Next Generation WYSIWYG HTML Editor - Froala](https://froala.com/wysiwyg-editor/)

[Quill - Your powerful rich text editor](https://quilljs.com/)

## EventSourcing in Ruby/Rails

[Ruby Event Store - use without Rails](https://blog.arkency.com/ruby-event-store-use-without-rails/)

[RailsEventStore/rails_event_store](https://github.com/RailsEventStore/rails_event_store/)

[Why I want to introduce mutation testing to the railseventstore gem](https://blog.arkency.com/2015/04/why-i-want-to-introduce-mutation-testing-to-the-rails-event-store-gem/)

[Mutation testing and continuous integration](https://blog.arkency.com/2015/05/mutation-testing-and-continuous-integration/)

[Rails Event Store - better APIs coming](https://blog.arkency.com/rails-event-store-better-apis-coming/)

[Event Sourcing is a transferable skill](https://blog.arkency.com/event-sourcing-is-a-transferable-skill/)

# Rake with Multiple Database

[Additional database-specific rake tasks for multi-database users](https://www.bigbinary.com/blog/rails-6-1-adds-additional-database-specific-tasks)

如果在 database.yml 中有設定 multiple database

那 db 操作的 rake 可以指定 database

例如

```elixir
rake db:migrate:primary
rake db:create:primary
rake db:drop:primary

rake db:migrate:ishin
rake db:create:ishin
rake db:drop:ishin
```

# Rails 7

[Rails 7.0 adds encryption to Active Record models](https://www.bigbinary.com/blog/rails-7-adds-encryption-to-active-record)

## CORS

[Rails CORS Guide: What It Is and How to Enable It](https://www.stackhawk.com/blog/rails-cors-guide/)

[Rails API & CORS. A dash of consciousness](https://thecodest.co/blog/rails-api-cors-dash-of-consciousness/)

[Rails Rack CORS](https://medium.com/nerd-for-tech/rails-rack-cors-6c0c7ee438bb)

[那些經歷過的 CORS 蠢問題](https://medium.com/@yovan/那些經歷過的-cors-蠢問題-e63576f67066)

rack

[https://github.com/cyu/rack-cors](https://github.com/cyu/rack-cors)

on nginx

[enable cross-origin resource sharing](https://enable-cors.org/server_nginx.html)