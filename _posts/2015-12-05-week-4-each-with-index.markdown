---
layout: post
title:  "Week 4 each_with_index"
date:   2015-12-05 10:07:48 +0000
categories: each_wtih_index
---
On Monday, I had my third session with my mentor.  We mainly talked about Object Oriented Programming.  We first talked about image_blur2.  My approach was ok, but there was room for improvement.  My mentor taught me a method called “each_with_index” in Ruby that could have shortened my code.  It is a handy method so I am writing the example usages here.  


* each_with_index with hash table.

{% highlight ruby %}

farm_animals = Hash.new
["pig", "cow", "horse", "sheep"].each_with_index do |animal, animal_index|
  farm_animals[animal_index] = animal
end

puts farm_animals.inspect

# this outputs:
# {0=>"pig", 1=>"cow", 2=>"horse", 3=>"sheep"}


{% endhighlight %}

* use with an array, taking advantage of index and find a location of horse

{% highlight ruby %}

["pig", "cow", "horse", "sheep"].each_with_index do |animal, animal_index|
  if animal == "horse"
    puts "horse is in index: #{animal_index}"
  end
end

# this outputs:
# horse is in index: 2


{% endhighlight %}

* and... use with two dimensional array (can find the positions of pig easily!)


{% highlight ruby %}

def index_test(array1)
    pigs = []
    array1.each_with_index do |row, row_index|
      row.each_with_index do |cell, column_index|
        if cell == "pig"
          pigs << [row_index, column_index]
        end
      end
    end
    return pigs
end


array1 = [
  ["pig", "bear"],
  ["dog", "cat"],
  ["cow", "pig"]
]

puts index_test(array1).inspect

# this outputs (where pigs are in array):
# [[0, 0], [2, 1]] 

{% endhighlight %}

Next we talked about image_blur3.  I don't want to go too much detail on how I did it, but I want to say that my image_blur3 code was very similar to my image_blur2 code, just with a few more lines.  I was delighted to hear that my mentor's approach was different from mine, because I love to hear different ways of thinking.  I got his code a couple of days later, and it was very interesting to see how he wrote it.  During our meeting, when he explained to me how he wrote his code using an absolute value with a distance and so on, I understood the concept but still wasn't clear on few things.  Looking at his code was very helpful for understanding his approach.  Also, I was amazed at how he efficiently he wrote his code.  I realized that I could write fewer lines of codes if I spent more time refactoring them.

Before the meeting, I spent some time looking around and figuring out the Linked List #1 problem.  I remembered Ken was talking about this during our office hour, so I looked at my notes and tried to understand what it was about.  Right before the meeting, while I was playing around my code, I accidentally got an answer, so I showed my mentor my code and discussed it with him.  He pointed that out my push() method was correct but pop() method was not right.  I somewhat understood why my pop() method was incorrect, but I felt like my pop() method was unnecessary to get the answer.  Anyway, I will spend more time on this code again to figure out what went wrong. 

I started working on Flixter (a Udemy clone) early in the week, but in the middle of the week, I decided to start over on the app.  I worked up to lesson 16 or so, but I forgot to implement TDD and I wanted to add the code while writing other code.  Writing testing code is a big challenge for me so I wanted to take my time working on it.  Next time I hope to write about my experience of writing testing code.   
