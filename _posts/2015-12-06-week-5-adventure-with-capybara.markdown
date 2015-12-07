---
layout: post
title:  "Week 5 adventure with Capybara"
date:   2015-12-06 16:07:48 +0000
categories: TDD, Capybara
---
When I was talking to my mentor early this week, I asked some questions about testing code.  I asked about what sort of testing I could be done in model and in controller for my Flixter project.  I also asked about integration testing, and he suggested I should research about it and write about what I learned.  

When I looked up the web, I learned that (if I understood correctly,) an integration test is a test for sequences of action, and it can be used to test a user flow or a user scenario.  When I searched 'integration test', I found many sites mentioned about a Gem called 'Capybara'.  I installed and tried to implement testing code using Capybara.  Here, I am going to write how I wrote a test suite with Capybara and what problems I encountered.  Once again, this is not a guide for how to use Capybara; rather, I am writing about my experience using it.  

First thing I did was to generate interation test file.

`rails generate integration_test user_flows`

In above command, "user_flows" was the name of the file I chose.  Running this command created user_flows_test.rb under test/integration/ folder.

Next, I added "capybara" in my Gemfile, and ran `bundle install`.

{% highlight ruby %}
gem 'capybara'
{% endhighlight %}  

In test/test_helper.rb, I added `require 'capybara/rails'` on the top of the file and also added `include capybara::DSL` inside `class ActionDispatch::IntegrationTest`.  Here's my test_helper.rb


{% highlight ruby %}
ENV["RAILS_ENV"] ||= "test"
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'
require 'capybara/rails'   #<---

class ActiveSupport::TestCase
  ActiveRecord::Migration.check_pending!

  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  #
  # Note: You'll currently still have to declare fixtures explicitly in integration tests
  # -- they do not yet inherit this setting
  #fixtures :all

  # Add more helper methods to be used by all tests here...
  def assert_attribute_is_validated(model, attribute, value, flag)
    model.assign_attributes(attribute => value)
    if flag == 1
      assert_not model.valid?
    else
      assert model.valid?
    end
  end

end

class ActionController::TestCase
  include Devise::TestHelpers
end

class ActionDispatch::IntegrationTest
  # Make the Capybara DSL available in all integration tests
  include Capybara::DSL   # <---

end
{% endhighlight %} 

I first tried to write a test to see if a user can successfully "sign up" and go to "My Dashboard" page.  This didn't take a long time for me to implement.  In test/integration/user_flows.test.rb, I added a test called "should create a new user"

{% highlight ruby %}

test "should create a new user" do
  visit root_path
  click_link "Sign Up"
  fill_in "Email", with: "penguin@example.com"
  fill_in "Password", with: "12345678"
  fill_in "Password confirmation", with: "12345678"
  click_button "Sign up"
  assert_equal root_path, current_path
  first(:link, "My Dashboard").click
  assert_equal dashboard_path, current_path 
end

{% endhighlight %} 

By looking at the above code, even if you never used Capybara, you sort of understood what was going on.  A user goes to a homepage, "root", and clicks "Sign Up" link, fills in email/password/password confirmation, and then clicks "Sign up" button.  This should take the user to a homepage again (assert statement).  If a user profile is successfully created, the user can go to "My Dashboard" page (another assert statement).  The last assert_equal should fail if a user profile is not created because "My Dashboard" link works only when the user signs in.  As you see, Capybara's commands are easy to understand.  The only thing I had an issue with this test was that I had two "My Dashboard" links on my homepage (one is at the top navigator place, and the other is on footer) and it gave me an error.  But I found a command, `first(:link, "My Dashboard").click`, solved a problem.

After successfully wrote the first testing code with Capybara, I was more confident and tried to test other user scenarios.  I tested to see "See All Courses" and "Learn More" links in my homepage could take me to courses_path/ directory.  They did.  Great.

{% highlight ruby %}

test "should go to list of all courses" do
  visit root_path
  click_link "See All Courses"
  assert_equal courses_path, current_path
end

test "learn more should go to list of all courses" do
  visit root_path
  click_link "Learn More"
  assert_equal courses_path, current_path
end

{% endhighlight %} 

Then, I tried to test if "Teach a Course" link works. Clicking "Teach a Course" should take to a "create a course" form.  This took me a long time to figure out.  Just like "should create a new user" test above, I tried to use `fill_in` command for "title", but it gave me an error saying "title is not found."  After one hour or so of struggle, I finally realized that a user was trying to create a course without signing in.  So, I thought I was in a course creation form, but I was actually in "sign in" page!  

I added a fake Email/Password again and tried to create a new course, a new section, and a new lesson with several assertion statements.  Here's long lines of test code I wrote:

{% highlight ruby %}
test "should create a course section lesson" do
    visit root_path
    click_link "Sign Up"
    fill_in "Email", with: "penguin@example.com"
    fill_in "Password", with: "12345678"
    fill_in "Password confirmation", with: "12345678"
    click_button "Sign up"
    assert_equal root_path, current_path
    click_link "Teach a Course"

    fill_in "Title", with: "lol"
    fill_in "Description", with: "omg"
    fill_in "Cost", with: "10"
    click_button "Create"
    assert page.has_content?('Add a new section...')

    click_button "Add a new section..."
    fill_in "section_title", with: "newsection"

    click_button "Add a section"
    assert page.has_content?('Add a New Lesson...')
    
    click_button "Add a New Lesson..."
    fill_in "lesson_title", with: "newlesson1"
    fill_in "Subtitle", with: "newlessonsubtitle1"

    assert page.has_content?('Add a New Lesson...')
end
{% endhighlight %}

I know above code doesn't look great, but I just started learning this and writing this itself was a big accomplishment for me.  I think I could probably insert FactoryGirl command in this test, but I wasn't sure how to use it.  Quite frankly, it took a long time to write this and I didn't have any energy left to refactor this code.

On the following day, when I read Capybara documentation, I found two very useful debug commands I didn't know on the previous day.  These commands made me able to see a snapshot of a current page.

`save_and_open_page`
(this would create a temporary html file in /tmp/capybara/ directory)   

`print page.html`
(this could give me a html statements in a screen and I could copy and paste to create a html file.)

If I knew these commands on the previous day, I didn't have to struggle when I was writing a test suite for checking "Teach a Course" link.  I should always read documentation carefully.  

I also wanted to test to see if a student can enroll the course.  I spent a long time adding the testing code for that, but I couldn't do it.  In Flixter, the student could use a credit card to enroll a course by going through a third party website.  I just couldn't figure out how to insert a fake credit card number information.  I decided this would be something I should be spending a time later, but not now when I am still new to Rails.  I read it somewhere maybe Mocha Gem might be useful this sort of situation.  Perhaps I could try to find about Mocha.  

It was a good experience trying Capybara even though I didn't get to implement all the testing scenerios I wanted to do.  I was glad I got to learn something new.  



