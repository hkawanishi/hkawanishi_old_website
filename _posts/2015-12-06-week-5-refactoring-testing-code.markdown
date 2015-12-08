---
layout: post
title:  "Week 5 refactoring testing code"
date:   2015-12-06 10:07:48 +0000
categories: TDD
---
As I wrote before, I stated my Flixter project last week but I decided to start over because I didn't implement any testing code.  Writing test suites is very challenging to me, so I wanted to take my time and practice.

In week 4, I tried adding some code in the controller test files.  This week, I tried to add some code in the model test files.  I watched [Codeschool's][codeschool] "Rails Testing for Zombies" videos (note: a subscription is required to watch these videos) and I tried to get a better grasp of testing code writing.  The videos seemed dated but I was still able to learn something from them.  Then, I tried to refactor my model testing code.  Obviously this is not "how to" guide of writing/refactoring testing code.  This is a record of what I learned and what I did.  

* This was my `test/model/course_model_test.rb`.  As you see, there were lots of repeated lines.

{% highlight ruby %}

require 'test_helper'
class CourseTest < ActiveSupport::TestCase
 

  test "title should be present" do
    course = FactoryGirl.create(:course)
    course.title = " "
    assert_not course.valid?
  end

  test "description should be present" do
    course = FactoryGirl.create(:course)
    course.description = " "
    assert_not course.valid?
  end

  test "cost should be present" do
    course = FactoryGirl.create(:course)
    course.cost = nil
    assert_not course.valid?
  end
  
  test "cost zero is valid" do
    course = FactoryGirl.create(:course)
    course.cost = 0 
    assert course.valid? 
  end

  test "free cost should not be negative" do
    course = FactoryGirl.create(:course)
    course.cost = -1 
    assert_not course.valid?
  end

  test "free cost is free" do
    course = FactoryGirl.create(:course)
    course.cost = 0
    assert course.free? 
  end

  test "premium class is not free" do
    course = FactoryGirl.create(:course)
    course.cost = 40
    assert course.premium? 
  end

end
{% endhighlight %}

* I first created the setup method on the top and moved the `@course = FactoryGirl.create(:course)` line inside it.  Then, I created the `assert_attribute_is_validated` method and also moved it to the top.  My code looked a bit better...

{% highlight ruby %}

require 'test_helper'
class CourseTest < ActiveSupport::TestCase

  def setup
    @course = FactoryGirl.create(:course)
  end

  def assert_attribute_is_validated(model, attribute, value, flag)
    model.assign_attributes(attribute => value)
    if flag == 1
      assert_not model.valid?
    else
      assert model.valid?
    end
  end

  test "title should be present" do
    assert_attribute_is_validated(@course, :title, " ", 1)
  end

  test "description should be present" do
    assert_attribute_is_validated(@course, :description, " ", 1)
  end

  test "cost should be present" do
    assert_attribute_is_validated(@course, :cost, nil, 1)
  end
  
  test "free cost should not be negative" do
    assert_attribute_is_validated(@course, :cost, -1, 1)
  end

  test "cost zero is valid" do
    assert_attribute_is_validated(@course, :cost, 0, 2)
  end

  test "free cost is free" do
    @course.cost = 0
    assert @course.free? 
  end

  test "premium class is not free" do
    @course.cost = 40
    assert @course.premium? 
  end

end

{% endhighlight %}

* Then, I learned that I could move the `assert_attribute_is_validated` method inside `test/test_helper.rb`, because this file is called every time I run the `rake` test.  
This further reduced the amount of lines in `test/model/course_model_test.rb` 

{% highlight ruby %}

require 'test_helper'
class CourseTest < ActiveSupport::TestCase

  def setup
    @course = FactoryGirl.create(:course)
  end

  test "title should be present" do
    assert_attribute_is_validated(@course, :title, " ", 1)
  end

  test "description should be present" do
    assert_attribute_is_validated(@course, :description, " ", 1)
  end

  test "cost should be present" do
    assert_attribute_is_validated(@course, :cost, nil, 1)
  end
  
  test "free cost should not be negative" do
    assert_attribute_is_validated(@course, :cost, -1, 1)
  end

  test "cost zero is valid" do
    assert_attribute_is_validated(@course, :cost, 0, 2)
  end

  test "free cost is free" do
    @course.cost = 0
    assert @course.free? 
  end

  test "premium class is not free" do
    @course.cost = 40
    assert @course.premium? 
  end

end

{% endhighlight %}

A portion of my `test/test_helper.rb` is shown below.  

{% highlight ruby %}

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

{% endhighlight %}
I also learned how to run an individual test file just by typing `ruby -Itest [testfile.rb]`
For example, if I just want to run course_test.rb, I type:

`ruby -Itest test/models/course_test.rb`


[codeschool]: https://www.codeschool.com/



