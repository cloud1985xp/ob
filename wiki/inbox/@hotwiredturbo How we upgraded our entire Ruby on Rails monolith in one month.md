---
title: "@hotwired/turbo: How we upgraded our entire Ruby on Rails monolith in one month"
source: "https://jonathanloos97.medium.com/hotwired-turbo-how-we-upgraded-our-entire-ruby-on-rails-monolith-in-one-month-25d110871039"
author:
  - "[[Jonathan Loos]]"
published: 2023-04-05
created: 2026-05-05
description: "More"
tags:
  - "clippings"
---
## Intro

Over the last 7 months, [we](https://harled.ca/team) have completed several large-scale upgrades to our [Rails Monolith](https://harled.ca/blog/our_majestic_monolith_part_one) in hopes of keeping our platform modern and ready for the next big move. The most daunting of which is undoubtedly the migration from [Turbolinks](https://github.com/turbolinks/turbolinks) and [@rails/ujs](https://guides.rubyonrails.org/working_with_javascript_in_rails.html#replacements-for-rails-ujs-functionality) to [@hotwired/turbo](https://github.com/hotwired/turbo) after we moved to Rails 7. Over the course of one month we overhauled nearly every corner of the UI & refactored most of our controller responses to accomplish the migration.

Please make sure to read through what these packages have to offer before continuing in the article as we’ll be focusing primarily on the upgrading process.

## Strategy

We started by grouping work in buckets of difficulty. After reading through the [turbo documentation](https://turbo.hotwired.dev/handbook/introduction) we found the following strategy would work well for us:

- Refactor the `method: :delete` to `turbo_method: :delete` for `link_to ` tags.
- Order all.js.erb (UJS) files by length to assess difficulty. With more lines came more reactivity and thus more complexity and refactoring needed. Those working on smaller files should be moving quickly through the list, with the higher complexity files taking longer to complete.
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*SMvqAekOjQPG78kj)

- We then chose to refactor all occurrences of `remote: true` into turbo frames because we wanted to start working with forms. (P.s we really found the form replacement lifecycle to be a great place to start learning about turbo!)
- Approach the rest of the files from short to long in small teams until the files remaining hits zero. For reference, we had to replace 177.js.erb files during our upgrade 🙃.

## Our approach to view components

For years we have been loosely adopting Joel Hawksley’s [View Components](https://viewcomponent.org/) into our app in hopes to eventually remove most of our partials! Although we still have a long way to go, during our migration to turbo we finally refactored hundreds of our partials into view components. To learn more of the benefits we see with moving to components [read here](https://jonathanloos97.medium.com/githubs-view-components-2ddbc3e532b1).

## Main fallout after migrating

### Same route being hit by multiple unknown sources

The simplest endpoint is one that has a single response as outlined in [Turbo’s handbook](https://turbo.hotwired.dev/handbook/frames). For example, consider an app with a users table.

User form component

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*wbgZMbrHP3d46ZiG)

Controller “update” action

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*kI4tsdPp6XxLKWD6)

HTML response

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*q8Z2menNU1oxkMiu)

This is a very simple lifecycle of a turbo-frame which is rendered inside a view component. But what if there was somewhere else in our app calling the same endpoint but requires a different response? One way we implemented this over the years (pre-turbo) is to do the following:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*HM3u-NLBxbQOs9GT)

**Challenge**

One of the main issues which we introduced gradually over the years was complex controller return methods (opposing the recommended [skinny controller](https://rails.rubystyle.guide/#skinny-controllers)). Some of our controller methods housed multiple responses conditionally rendered based on large amounts of business logic. This was an obvious growing pain of our platform, and we were excited / terrified at the opportunity to detangle them! We determined there were a couple approaches to the issue:

1. Leave the refactoring of the controller to another time and rebuild the response partials with turbo-frames as-is.
2. Clean up the controller method to have one response turbo\_stream and have the conditional logic in the partial.
3. Break up the controller method for each response with a dedicated endpoint for each.

We chose to pursue the third option. While it introduces a bit more risk to refactor our app that heavily, we determined the benefit of simplifying our controller interface was worth time. The resulting controller from the previous example would be:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*qbtLDtM5kfm9jRUv)

### Error handling

We ran into multiple instances where error cases were handled incorrectly as a result of the move which left users confused as to whether their changes were persisted or not. The main items we identified were:

- Forms not properly displaying the error messages. For example, if an endpoint didn’t properly handle a failed update and defaulted to rendering the read-only component instead of responding with the form & errors.
- Toast messages not set on redirects.
- For async queries triggered through stimulus controllers: Uncaught errors leading to console error logs.

### Missing form fields/attributes lost during move to component

As I mentioned a large part of our upgrade was also moving a lot of our views into view components. We had a lot of success with this, but we did have one big issue: human error. We were our own worst enemies and missed some fields/UI components during the move to components. This is mostly a friendly reminder to take your time while you’re refactoring your code, it’s easy to miss something! Also note that we have very little end-to-end tests, so we had to do most of the validation manually.

This brings up a good point, due to the lack of end-to-end testing we had to make an elaborate manual tracker to ensure we covered the entirety of the application before the final merge. Whatever stage your testing strategy is at, it’s always prudent to consider and establish a testing strategy prior to beginning implementation. In our case it was manual acceptance testing, how are you planning on doing it?

## Upgrading to 7.3.0

This is mostly an announcement for anyone who is running @hotwired/turbo < 7.3.0 and looking to update the minor version. There are a couple intricacies you should be aware of which are outlined in [this github thread](https://github.com/hotwired/turbo/pull/863#issuecomment-1490066773). This upgrade rendered most of our application unusable without the incorporation of a temporary patch.