---
layout: post
title:      "Things you may have missed in flatiron schools software engineering course"
date:       2019-08-22 17:56:34 -0400
permalink:  things_you_may_have_missed_in_flatiron_schools_software_engineering_course
---


## Welcome to Flatiron School

- How do I contact Flatiron about _________ ?
  
  [https://flatironschool.com/contact-us](https://flatironschool.com/contact-us)

- I got accepted - How do I join my cohorts slack channel and opening video conference?

  It may just take some time, to get added to slack and notified when you'll meet. otherwise, you can contact your cohort lead or [admissions](mailto:admissions@flatironschool.com).

## Getting support

- Stuck? Don't understand somthing? There are generally three places to go for help google, Ask A Question (AAQ) and the slack channels.

- Still don't having issues? ask again!

- Don't forget to ask. If your searching for a solution for more than an hour, you should probably just ask.

## learn from the command line (local setup)

- How do I open a lesson from the command line?

  `learn open` opens the lesson your currently up to.
  
- If you're in the middle of a lesson you can use `learn next` to open the next lesson.

  `learn open lab-name-here` will open that lab, this can be with or without the section/cohort part of the URL:

  `learn open ruby-variable-assignment`
  
  `learn open ruby-variable-assignment-bootcamp-prep-000`

- When using the local environment how can I customize what editor learn will open?

  You can change the editor option in the `~/.learn-config` file. Examples include:
  
  Visual Studio Code: `code`; Sublime Text: `subl`; XCode: `xed`; Atom: `atom`.

## Debugging learn command issues

- Can you connect to and authenticate with learn's servers correctly?
  
  check that running `learn whoami` (who-am-i) prints your info and that `learn hello` prints "Hello {your first name}".

- Trying to open a lab and getting an error similar to ``/gems/learn-open-1.2.22/lib/learn_open/lessons/ios_lesson.rb:6:in `detect': undefined method `any?' for false:FalseClass (NoMethodError)``?

  The lesson you're trying to open is missing the .learn file which combined with [issue #23](https://github.com/learn-co/learn-open/issues/23) is causing this error.

  As a workaround, you can git clone the repo and bundle/npm install.

- If you're having issues with the running the tests try running the tests directly `rspec spec` on ruby labs and `npm test` on javascript labs.

- issue submitting? `learn submit` doesn't do any magic. `git add .`, `git commit -m "Done"`, `git push` and creating a pull request on GitHub, will accomplish the exact same thing, including marking the lab as done.

## Ruby

- I find myself writing to many nested if statements. is there an easier way to deal with calling methods on optional properties?

  ```ruby
  if user.profile
    if user.profile.photo.present?
      # display user profile photo
    else
      # display generic profile photo
    end
    # display generic profile photo
  end
  ```

  Can be shortened and [DR(Y)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)ied using the [Safe Navigation Operator](https://ruby-doc.org/core-2.6/doc/syntax/calling_methods_rdoc.html#label-Safe+navigation+operator) added in ruby 2.3.0:

  ```ruby
  if user.profile&.photo.present?
    # display user profile photo
  else
    # display generic profile photo
  end
    ```

- I uploaded my rails project to Heroku and My images aren't working anymore.

  Filenames get a hash added during the production build step, so their URL can't be hardcoded. instead use the asset_path helper to get the correct URL `<%= asset_path 'logo.png' %>`. More on that in the [asset pipeline docs](https://guides.rubyonrails.org/asset_pipeline.html#coding-links-to-assets). This also works for [.css.erb files](https://guides.rubyonrails.org/asset_pipeline.html#css-and-erb) and similarly in [.sass files](https://guides.rubyonrails.org/asset_pipeline.html#css-and-sass).

## Blog
  
- How can I revert changes to my blog?

  Your blog is stored in a GitHub repository at `https://github.com/{your-name}/{your-name}.github.io` so you can always use git to get any previous state.

- How do I change the theme?

  The blogs use [Jekyll](https://jekyllrb.com).

## I made an extension for learn.co please give it a try

- I often struggle with labs only to later find out that there were already many reported issues on Github.

  I made an extension to solve just that problem. You can get it at <https://github.com/arye-dov-eidelman/learn-issues-extension>.

