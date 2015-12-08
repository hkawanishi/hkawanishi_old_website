---
layout: post
title:  "Week 5 adding validations to lessons and sections"
date:   2015-12-06 11:07:48 +0000
categories: Flixter, validations
---
While I was working on my Flixter, a Udemy clone, I decided to add input validation code for lessons and sections.  I wanted to make it so when a teacher wanted to create a lesson or a section, it would warn if there was no title or subtitle.  There were instructions on how to add validations to create a course so I thought it was easy.  

In Flixter's model/course.rb, I saw the following three lines:

{% highlight ruby %}

validates :title, :presence => true
validates :description, :presence => true
validates :cost, :presence => true, :numericality => {:greater_than_or_equal_to => 0}

{% endhighlight %} 

and in controllers/instructor/courses_controller.rb, I saw the following lines:

{% highlight ruby %}

def new
  @course = Course.new
end

def create
  @course = current_user.courses.create(course_params)
  if @course.valid?
    redirect_to instructor_course_path(@course)
  else
    render :new, :status => :unprocessable_entity
  end
end

{% endhighlight %} 

So, it looked like I could add similar code to my lesson and section files.  However, it wasn't as easy as I thought.  After moving my lesson and section forms into modal, the `def new` method was deleted from controller files so it wouldn't render correctly when lesson/section entries were invalid (see above code).  I talked about the problem with Ken but he said it was very challenging to do what I wanted to do.  However, he suggested a workaround.  He said I could "create a 'new' form that's actually only used on validation errors."  I took his advice and created an error page.  I am describing here what I did.  I am only going to write how I changed the lesson files but the section files were done in the same way.

In my model/lesson.rb, I added:

{% highlight ruby %}

validates :title, :presence => true
validates :subtitle, :presence => true

{% endhighlight %} 

and then, in config/routes.rb, I changed a line, adding :new for lesson.

{% highlight ruby %}
resources :courses, :only => [:index, :show] do
    resources :enrollments, :only => :create
  end
  resources :lessons, :only => [:show]
  namespace :instructor do
    resources :lessons, :only => [:update]
    resources :sections, :only => [:update] do
      resources :lessons, :only => [:new, :create]  # this line.
    end
    resources :courses, :only => [:new, :create, :show] do
      resources :sections, :only => [:new, :create]
    end
  end
{% endhighlight %} 

In my controllers/instructor/lessons_controller.rb, I added 'new' and edited 'create'

{% highlight ruby %}

def new
end

def create
  @lesson = current_section.lessons.create(lesson_params)
  if @lesson.valid?
    redirect_to instructor_course_path(current_section.course)
  else
    render :new, :status => :unprocessable_entity
  end
end

{% endhighlight %} 

And, I created the new.html.erb file under views/instructor/lessons/

{% highlight html %}
{% raw %}

<br />
<div class="booyah-box col-xs-10 col-xs-offset-1">
<h3>Please make sure to you have a <span class="entryvalidation">"title"</span> and a <span class="entryvalidation">"subtitle"</span> when creating a new lesson</h3>
<br />
 <%= link_to 'Go back to my course to create a lesson again', instructor_course_path(current_section.course) %>
</div>

{% endraw %}
{% endhighlight %}

If a teacher doesn't enter a title or a subtitle of a lesson, this page is displayed along with a link to go back to the course page.  I also made a minor change to master.css.scss to make the words "title" and "subtitle" appear in red color.

{% highlight css %}
.entryvalidation {
  color:red;
}
{% endhighlight %} 

The error message page I made is shown below.  
<figure>
  <a href="/images/lesson_error_page.png"><img src="/images/lesson_error_page.png"></a>
  <figcaption>The error message page (new.html.erb)</figcaption>
</figure>

Someday when I become much better at web development, I want to be able to add better validation code and avoid work-arounds.  But for now, this will do.




