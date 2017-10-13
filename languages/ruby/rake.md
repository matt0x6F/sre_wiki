Using Rake to Automate Tasks
====

Posted in tools with tags rake ruby - Friday, July 1, 2016

### Summary

Rake enables you to define a set of tasks and the dependencies between them in a file, and then have the right thing happen when you run any given task. Each task may be either one of the built-in types, or a block of your own Ruby code. It was originally created to handle software build processes, but the combination of convenience and flexibility that it provides has made it the standard method of job automation for Ruby projects.

## Writing Rake Tasks

You may use supplied methods in your task code to conveniently set up common jobs, such as running test suites, publishing files and packaging software. Equally, you may call or write any other Ruby code into a Rake task, which means that it can automate just about anything.

Crucially, you also write the task and dependency definitions themselves in Ruby, following a specific format. This means that you do not need to deal with any new syntax to start automating your routine jobs. Anyone with a basic knowledge of Ruby can understand and maintain Rake tasks.

### Rake is Declarative

Rake itself is designed to be a declarative system - you specify the result that you want, and Rake carries out the associated task and dependent tasks as necessary. This means that a set of correctly defined tasks will do as little or as much as is necessary to produce a known state.

The task types that are supplied with Rake follow this approach. For example, the built-in types of tasks for creating file and directories creation automatically check for the specified item, and will not run if an up to date copy exists.

### Project and Global Rake Task Files

By default, the Rake utility checks the current working directory for a file with the name _Rakefile_ (with no extension). This enables you to add a file of Rake tasks to the source code of any application without conflicting with the names of existing files. If you would like to create one or more specifically named Rake files in a directory, use the file extension .rake for them.

To make a set of Rake tasks available for use from any directory, create a _.rake_ subdirectory within your home directory, and place the appropriate Rake files there. Any rake command with the _-g_ option will use these global Rake files:

`rake -g -T`

## Installing Rake

Rake is now part of the Ruby standard library, and will automatically be part of any modern Ruby installation. Alternative Ruby implementations such as JRuby usually also include Rake.

To install Rake for Ruby 1.8, use RubyGems:

`gem install rake`

## Using Existing Rake Tasks

Many Ruby projects and applications provide a set of Rake tasks, so you may well start using Rake before you have written a task file yourself. For example, every Ruby on Rails project automatically includes a large number of tasks which can be run from the root directory of the project.

### Getting a List of the Available Tasks

To see a list of all of the tasks available from the current Rake file, use the -T option of the rake utility:

`rake -T`

### Running a Task

To run a named task, specify the name of the task:

`rake my_task`

Notice that the task name has no colon prefix here.

To run a Rake file or directory task, use the name of the file or directory:

`rake mydoc.pdf`

If you call _rake_ without specifying any task, it automatically checks for a task named default, and runs that task if one is found.

`rake`

### Specifying Options

All of the options of the rake utility may be called with either a single letter switch, or a longer word version. Note that command-line options may go before or after the name of the task, whichever you prefer:

`rake my_task —quiet`
`rake —quiet my_task`

The –quiet option suppresses any output that individual tasks would usually display in your terminal window, but allows errors to be shown normally.

A later section explains some of the most used options. To see a list of all of the available options, run _rake_ with -h, or –help:

`rake -h`
`rake —help`

## Writing Rake Files

A Rake file does not need to have anything in it, other than task definitions. In addition to the tasks, you may freely use standard Ruby elements, including constants and methods.

Note: _Remember to set a default task._ It’s a good habit to specify a default task in each Rake file (as explained below).

### Task Definitions

Each task definition consists of:

* A description
* The name that identifies the task
* The code to be executed by the task

In addition, you can specify input parameters for your tasks, and other tasks that are prerequisites.

For standard tasks, the name of the task is a Ruby symbol, which means that it must be prefixed by a colon, and also must use only lowercase alphanumeric characters, with underscores instead of spaces. We only use the colon prefix for specifying task names within Rake files, not when we actually run the tasks.

Note: _Multiple Tasks with the Same Name._ If you define two tasks with same name, Rake appends the second task of that name to the first.

The actual code for the task to run must be enclosed in a standard Ruby do…end block. This is the structure of a simple task:

```ruby
desc "One line task description"
task :name_of_task do
  # Your code goes here
end
```

The task shown above accepts no input parameters, and has no prerequisites, so we completely omitted these items to make the definition more compact. If we did not include a description, the task would still work, but it would not appear on listings.

This task depends on two other tasks:

```ruby
desc "Example of a task with prerequisites"
task :third_task => ["first_task", "second_task"] do
  # Your code goes here
end
```

When we run the task shown above, Rake first checks the prerequisites. It then executes all of the prerequisite tasks before it actually carries out the requested task.

To run a task, we simply call the rake utility at a command prompt, and specify the task:

`rake name_of_task`

Note: _The Initial Working Directory._ To ensure that paths are consistent, all tasks run with an initial working directory that matches the directory of the Rake file that holds them. Do not assume that the working directory of a task is the current working directory of the user that runs the task.

### Input Parameters for Tasks

Rake refers to the input parameters for tasks as arguments. This task has both arguments and prerequisites:

```ruby
desc "Example of task with parameters and prerequisites"
task :my_task, [:first_arg, :second_arg] => ["first_task", "second_task"] do |t, args|
  args.with_defaults(:first_arg => "Foo", :last_arg => "Bar")
  puts "First argument was: #{args.first_arg}"
  puts "Second argument was: #{args.second_arg}"
end
```

The first line of the task assigns default values to the arguments.

To run a task that requires arguments, we must specify the values for each of the arguments after the name of the task:

`rake my_task[one,two]`

Notice that there are no spaces where the task is specified, either between the name of the task and the opening square bracket, or between the first and second argument.

If you do not give a value for an argument at the command-line, and the task does not specify a _default_ value, the value of the argument is set to _nil_.

### Setting a Default Task

For convenience, define a dummy task called _default_ that depends on one or more of your tasks, and has no code itself. If a user runs the _rake_ utility without specifying a task, it automatically runs the _default_ task in the Rakefile.

`task :default => ["my_task"]`
`
This command now carries out the default task:

`rake`

### Running One Task with Another

Normally, you link tasks together with dependencies. If you need to run a task from within another, use the invoke method:

```ruby
desc "Example of task with invoke"
task :first_task do
  Rake::Task[:second_task].invoke
end
```

This code returns the specified task as a Rake::Task, and calls the invoke method of the task object.

## Managing Files and Directories with Rake

### File and Directory Generation Tasks

File and directory creation tasks are identified by the item that they generate, rather than by an arbitrary name. Any file or directory task may use either full or partial names. Note that you must use glob patterns to define any partial names, not regular expressions. This provides consistency with the standard Ruby file and directory functions, which also use glob patterns.

```ruby
file 'mydoc.pdf' => ['mydoc.xml', 'mydoc.xslt'] do
  # Code goes here...
end
```

If a task specifies a target file, then Rake checks both that the file exists, and also that it is not older than the files specified by any prerequisite tasks. Rake only runs the task associated with the file if the target is either not present, or if it is not up to date. As a result, Rake tasks can efficiently maintain even a very large set of files.

Directory tasks just create the specified directories, if they do not already exist. These consist of the keyword directory, followed by the directory itself. Directory tasks may not have either a code block, nor any prerequisites. Other tasks may use a _directory_ task as a prerequisite:

```ruby
directory 'html'

file 'html/contents.html' => ['html', 'html/index.html'] do
  # Code goes here...
end
```

As you would expect, standard tasks may have file or directory tasks as prerequisites:

```ruby
task :upload_page => 'myfile.html' do
  # Your code...
end
```

Rather than tying tasks to specific file or directory names, you may use FileLists or Rules, as explained below.

### Defining Sets of Files with FileLists

A FileList is an array of complete and partial file names, written in the Rake file itself (or one of the imported modules).

```ruby
my_files = FileList['build/*.html', 'index.xml']
```

Note: _Excluded Files._ By default, FileLists never returns certain types of file, even if a glob would match them. Suppressed file types include administrative files and directories for version control systems, UNIX backup files, and core dumps.

### Using the Clean-up Tasks

Rake includes two tasks to clean up a set of files, so that you do not need to write code to handle this kind of job. Simply require the rake/clean module, and add values to the FileLists called _CLEAN_ and _CLOBBER_, which are part of the module. Any file that is in the _CLEAN_ FileList will be deleted when you run the _clean_ task. Similarly, _clobber_ deletes anything included in the _CLOBBER_ list.

```ruby
require 'rake/clean'
CLEAN.include('*.tmp')
CLOBBER.include('*.tmp', 'build/*')
```

Having two tasks with separate FileLists enables you to target either just those files that were created by running build tasks, or purge any file that should not be mixed in with the set. Set up the _CLEAN_ list to specifically handle temporary build files, and _CLOBBER_ to aggressively match all potentially unwanted files.

### Other File Handling Methods

Rake automatically provides file handling methods, to enable you to write tasks that include the usual operations. For convenience, these methods have similar names to the equivalent UNIX utilities, such as cp and mv.

Here are two contrived examples:

```ruby
task :copy_files do
  cp('readme.htm', File.join('build', 'readme.htm'), :verbose => true)
end

task :mv_files do
  mv(File.join('build', 'readme.htm'), File.join('release', 'index.htm'), :verbose => true)
end
```

These methods are provided by _RakeFileUtils_, an extended version of the standard Ruby module _FileUtils_. To see all of the available methods, refer to the _FileUtils_ documentation:

`ri FileUtils`

Use the standard tasks where possible, rather than these utility methods.

### Rules

A rule defines filenames, and a block of Ruby code. For each matching file, the rule creates and runs a new task with the specified code. You may use either names or regular expressions to define the files that match the rule.

## Using Other Ruby Libraries in Rake Tasks

Your Rake files may reference other Rake files or Ruby modules. Use _require_ statements as normal to import modules, or other Rake files.

Note: Rake includes built-in methods for many common tasks, so before you write or import other code, check **the documentation** to see if a suitable method already exists within Rake.

This example uses the ERB template system, which is part of the Ruby standard library:

```ruby
require 'erb'

OUTPUT_FILE='README.html'
TEMPLATE_FILE='template.html.erb'

def get_template
  File.read(TEMPLATE_FILE)
end

desc "Builds the HTML file, using ERB."
file OUTPUT_FILE do
  File.open(OUTPUT_FILE, "w+") do |f|
    f.write(ERB.new(get_template).result())
  end
end

task :default => [OUTPUT_FILE]
```

The code shown above uses a standard Ruby method from the _File_ class to get the template from a file as a string, creates an ERB renderer to process the template string, and writes the output to the target file.

## Integrating with the Shell

Rake and Ruby work consistently across platforms, making them a portable alternative to shell scripts, but you may also use them as a means of automating platform-specific features. This section briefly describes how you can use the shell integration facility of Rake to call command-line utilities from within your own tasks.

### Using Shell Commands in Rake Tasks

To handle shell commands in your Rake tasks, use the _sh_ method. This is provided by an extension to the _FileUtils_ module, so you need to require that module in the task file before you can call the method:

```ruby
require 'fileutils'

# Stuff...

task :run_command do
  sh %{ space separated command and options }
end
```
The _sh_ method passes two outputs: a status, and the actual output of the command. This example from the Rake documentation makes it clear:

```ruby
sh %{grep pattern file} do |ok, res|
  if ! ok
    puts "pattern not found (status = #{res.exitstatus})"
  end
end
```

As always, remember that using shell commands restrict your software to systems that have those exact utilities installed. Use Ruby code unless you require the features of a particular command-line utility, such as a configuration tool that is specific to one operating system.

### Using Environment Variables

Rake automatically recognises the environment variables provided by the shell. You may also create or set environment variables specifically for tasks at the same time as you run Rake.

This code uses two environment variables, the standard HOME variable, and a MY_VAR that was set at run-time:

```ruby
desc "Task description"
task :name_of_task do
  my_setting1 = ENV['HOME']
  my_setting2 = ENV['MY_VAR']
  # Your code goes here
end
See below for information on how to set variables at run-time.
```

### Outputting Messages to the Shell

By default, Rake echoes error messages and the file handling operations of RakeFileUtils to the shell. You can use standard Ruby methods to output your own messages as normal:

```ruby
desc "Task description"
task :name_of_task do
  puts "foo bar"
end
```

Your output appears even if Rake runs with the –quiet or –silent options.

## Managing Large Numbers of Tasks

If the number of required tasks in a set begins to grow, you can ensure that they remain easy to use and maintain with two techniques. First, you can assign some of your tasks to separate _namespaces_, so that task names remain unique and unambiguous. Second, you can write code into the Rake file to dynamically generate tasks, enabling it to create many variations of the same process, or adapt the available tasks to the host system.

### Using Namespaces to Organize Tasks

To avoid duplicate or ambiguous names in large sets of tasks, enclose some of the tasks in namespaces. By convention, each namespace should be identified by one word, in lowercase. You may nest your namespaces, as this example shows:

```ruby
namespace 'build' do

  # tasks...  

end

namespace 'test' do

   # tasks...

  namespace 'unit' do
   # tasks...
  end

end
```

To reference a task that is within a namespace, prefix the name of the task with the namespace and a colon:

```ruby
desc "Example of a task with namespaced prerequisites"
task :third_task => ["build:first_task", "test:unit:second_task"] do
  # Your code goes here
end
```
The command-line utility uses the same naming convention:

```
localhost:railsapp me$ rake -T
(in /Users/me/railsapp)

rake rails:freeze:edge # Lock to latest Edge Rails
rake rails:freeze:gems # Lock this application to the current gems
rake rails:template # Applies the template supplied by LOCATION
rake rails:unfreeze # Unlock this application from freeze of gem
```
The above was taken from the task list of a standard Ruby on Rails application.

### Defining Tasks Dynamically

To dynamically generate task definitions as the task file runs, directly integrate the definition keywords into Ruby code. All of the Rake keywords are actually Ruby methods that declare tasks, descriptions and namespaces as the tasks file is executed.

The example shown below dynamically creates a suite of tasks by loading YAML format configuration files from a directory. It produces a separate namespace for each configuration file.

```ruby
require 'yaml'

# Uses FileList to get an Array of the configuration files
CONFIG_FILES=FileList['config/*.yml']

# Returns the configuration from the file as a Hash object
def get_config(file)
  YAML.load_file(file)
end

CONFIG_FILES.each do |f|

  config = get_config(f)

  namespace config[:name] do

    # Generate tasks

    desc "First task for #{config[:name]}"
    task config[:first] do
      # Code goes here
    end

    desc "Second task for #{config[:name]}"
    task config[:second] do
      # Code goes here
    end

    # more...

  end

end
```

This approach enables us to cleanly generate multiple tasks with appropriate namespaces. If we only needed to generate a task for each file, we could make an even simpler routine by using a Rule to create the tasks.

## More Advanced Options for Running Rake Tasks

The _rake_ utility provides many options. Some of these options are useful in many situations for testing, or for working around issues with the host system.

### Specifying the Location of the Tasks File

To use a specific Rake file of your choice, use either -f, or –rakefile:

`rake --rakefile my_task_file my_task`

To use tasks from the Rake files from your _.rake_ directory, use the -g option:

`rake -g -T`
`rake -g my_task`

This option automatically uses all of the _.rake_ files in the directory.

### Suppressing the Output of Tasks

The –quiet option only suppresses normal output. Use –silent to run Rake with absolutely no output at all:

`rake --silent my_task`

###Specifying Environment Variables for a Task

To specify environment variables for your tasks along with your command, simply use the format variable=value:

`rake my_task my_var1='Some value' my_var2='Another value'`

### Debugging Tasks

Finally, to debug a Rake task, either use the –dry-run option to see what steps it will take, without actually executing them, or specify –trace to see detailed output:

`rake —-dry-run my_task`
`rake -—trace my_task`

Remember that Ruby itself includes a debugger that works on the command-line.

## Other Automation Tools

Beyond Rake, Ruby has a broad range of other useful automation software. These are a few of the most well-known:

* The [Thor](http://github.com/wycats/thor) scripting framework provides a powerful general-purpose system for defining and running tasks.
* [Capistrano](http://www.capify.org/) is the leading utility for automating server application deployments and upgrades.
* [Vlad](http://rubyhitsquad.com/Vlad_the_Deployer.html) is a rival to Capistrano which directly extends Rake.
* [Puppet](http://reductivelabs.com/products/puppet) enables you to automate the management of a network of varied systems, throughout their lifecycle.
* [Chef](http://wiki.opscode.com/display/chef/Home) provides an alternative to Puppet that is perhaps more suited to Ruby developers who must maintain application servers.

## Useful Resources

### Tutorials

* [Ruby on Rails Rake Tutorial](http://www.railsenvy.com/2007/6/11/ruby-on-rails-rake-tutorial)
* [The official Rake tutorial](http://docs.rubyrake.org/)

### Reference Documents

* [The official API documentation](http://rake.rubyforge.org/)
* [Martin Fowler’s guide to Rake](http://www.martinfowler.com/articles/rake.html)
* [Rake Quick Reference, by Greg Hudson](http://ghouston.blogspot.com/2008/07/rake-quick-reference.html)

### Hints and Tips

* [Jay Fields on Testing Rake Tasks](http://blog.jayfields.com/2006/11/ruby-testing-rake-tasks.html)
* [John Barnette on Rake](http://www.jbarnette.com/2009/08/27/on-rake.html)

Source: http://www.stuartellis.name/articles/rake/
