####Let’s install some Ruby!

[Windows][1]  

OS X:  
```
brew install ruby  
```
Linux:  
```
sudo apt-get install ruby1.9.1  
```

####Hello World

Create a text file called hello-world.rb containing the following code:  

```puts ‘Hello world’```

Now run it:  

```
ruby hello-world.rb  
```

You can also run the short “hello world” program without creating 
a text file. This is called a one-liner:  
```
Ruby -e "puts 'Hello world'"  
```
You can run this code with irb, but the output will look slightly different. puts will print out "```Hello world```”, but irb will also print out the return 
value of puts – which is ```nil```.  

run 
```
irb  
```
and enter the following commands:
```
>> puts “Hello world”  
Hello world  
=> nil  
```
####Syntax

Let’s say you have a class Person: 

```
class Person  
end
```  
```  
person = Person.new  
person.name # => no method error  
```  
Obviously we never defined method name. Let’s do that: 

```
class Person  
  def name  
    @name # simply returning an instance variable @name  
  end  
end
``` 
```  
person = Person.new  
person.name # => nil  
person.name = "Dennis" # => no method error  
```  
We can read the name, but that doesn’t mean we can assign the name. Those are two different methods. Former called reader and latter called writer. We didn’t create the writer yet so let’s do that. 

```
class Person  
  def name  
    @name  
  end  

  def name=(str)  
    @name = str  
  end  
end
```  
```  
person = Person.new  
person.name = 'Dennis'  
person.name # => "Dennis"  
```  
Awesome. Now we can write and read instance variable @name using reader and writer methods. But why waste time writing these methods every time?  

```
class Person  
  attr_reader :name  
  attr_writer :name  
end
``` 

Even this can get repetitive. When you want both reader and writer, just use accessor:  

```
class Person  
  attr_accessor :name  
end
```  
```  
person = Person.new  
person.name = "Dennis"  
person.name # => "Dennis"  
```  
```
class Person  
  attr_accessor :name  

  def greeting  
    “Hello #{@name}”  
  end  
end
```  
```  
person = Person.new  
person.name = "Dennis"  
person.greeting # => "Hello Dennis"  
```  
**Starting a new Ruby project:**
```  
cd ..  
mkdir Ruby_uva  
cd Ruby_uva  
```  
####BDD

Let’s create that Gemfile:  

```
source "http://rubygems.org"  

gem "rack"  
gem "cucumber”  
gem "rspec"
```  

Run ```bundle```  or ```bundle install```  

``` 
mkdir features  
mkdir features/step_definitions  
mkdir features/support  
vim features/student_can_submit_assignment.feature  
``` 

**The syntax of a Cucumber feature**

``` 
Feature: Student Can Submit Assignments  

As a student  
I can submit my assignment  
So I can prove my knowledge  

Scenario: Student can submit an assignment  
Given I am a student  
When I submit an assigment to my teacher  
Then my teacher should have my assignment  
``` 

Run ```bundle exec cucumber``` 

in ```features/step_definitions```, add ```students_assignment_steps.rb``` and paste in what your terminal so nicely provided you with  

```
Given /^I am a student$/ do  
pending # express the regexp above with the code you wish you had  
end  

When /^I submit an assignment to my teacher$/ do  
pending # express the regexp above with the code you wish you had  
end  

Then /^my teacher should have my assignment$/ do  
pending # express the regexp above with the code you wish you had  
end  
```

Run ```bundle exec cucumber``` again and you'll see that because your ```Given``` is pending, it won't even bother to run your ```When``` and ```Then```. Change your ```Given``` in:  

```
Given /^I am a student$/ do  
  @student = Student.new  
end  
```

Run ```bundle exec cucumber``` and it will give you Run ```uninitialized constant Student```. We need another directory:  

```
mkdir lib  
vim lib/student.rb  
```

Paste in: 
```
class Student  
end  
```

In order for Cucumber to know about this new directory, open ```features/support/``` and add a document called ```load_em.rb```.
Paste in  

```Dir[File.dirname(__FILE__) + "/../../lib/*.rb"].each{|f| require f}```  

Run ```bundle exec cucumber```  and you'll see ```Given I am a student``` is now green.  
Now that we've created the framework in Cucumber, we can go ahead and write our code TDD-style, writing tests in rspec prior to our controllers.  

In ```students_assignment_steps.rb``` fill in your ```When```:  

```
When /^I submit an assignment to my teacher$/ do  
  @assignment = Assignment.new  
  @teacher.submit_assignment(@student, @assignment)  
end  
```

Create ```lib/assigment.rb``` and paste in:  

```
class Assignment  
end  
```

Open ```student.rb``` and add:  

```
def submit_assignment(teacher, assigment)  
end  
```

under ```class Student```. When running ```bundle exec cucumber```, your test will return green.  
In ```students_assignment_steps.rb``` fill in your ```Then```:  

```
Then /^my teacher should have my assignment$/ do   
  @teacher.assignment_for_student(@student).should eq(@assignment)  
end  
```

To solve the ```undefined method 'assignment_for_student'``` we need to add a teacher.  

Add  
```
Given /^I am a student$/ do  
  @student = Student.new  
  @teacher = Teacher.new  
end  
```

to your ```students_assignment_steps.rb```  and create a ```teacher.rb``` in your ```/lib``` folder:  

```
class Teacher  
 def assignment_for_student(student)  
  end  
end  
```

We added ```def assignment_for_student(student)``` so you could avoid running Cucumber again and figuring out we don't have assignments yet either.  

**Now we're all ready do touch some rspec.**  

```
mkdir spec  
vim spec/teacher_spec.rb  
```

In ```teacher_spec.rb``` write:  

```
require_relative "../lib/teacher"  
require "rspec"  

describe Teacher do  
  it "should store assignments" do  
    student = stub  
    assignment = stub  
    subject.submit_assignment(student, assignment)  
    subject.assignment_for_student(student).should eq(assignment)  
  end  
end  
```

Open ```teacher.rb``` and add  

```
def initialize  
  @assignments = {}  
end  
def submit_assignment(student, assignment)  
  @assignments[student] = assignment  
end  
```

under ```class Teacher```, and change  

```
def assignment_for_student(student)  
end  
```

in

```
def assignment_for_student(student)  
  @assignments[student]  
end  
```

When we run ```bundle exec cucumber``` we see some greens, some reds. Now would be a good time to initiate a git repository. Run:  

```
git init  
git status # to see a list of our untracked files  
git add .  
git commit -m "Add: Student can submit assignment"  
```

**Let's add another feature!**
Create:  
```vim features/teacher_can_grade_assignment.feature```  

Open the feature file and write:  
```
Feature: Teacher can grade assignment  

As a Teacher  
I can grade my students' assignments  
So that they can know their knowlede level  

@wip
Scenario: Teacher can grade assignment  
  Given I have a student  
  And They submitted an assignment  
  When I grade the assignment  
  Then the assignment has a grade  
```

When we run ```bundle exec cucumber --tags @wip``` we avoid running both scenarios, as that might get noisy. 

Create ```features/step_definitions/teacher_grade_assignment.rb```. Pasting in the step definitions our terminal provides us with again, we can also add some classes already:  

```
Given /^I have a student$/ do  
 @teacher = Teacher.new  
 @student = Student.new  
 @assignment = Assignment.new  
end

Given /^They have submitted an assignment$/ do
  @teacher.submit_assigment(@student, @assignment)
end

When /^I grade the assignment$/ do
  @teacher.record_grade(assignment, :a)
end

Then /^the assignment has a grade$/ do
end

```




[1]: http://rubyinstaller.org/
