# Shell

Shell implements an idiomatic Ruby interface for common UNIX shell
commands.

It provides users the ability to execute commands with filters and
pipes, like `sh`/`csh` by using native facilities of Ruby.

## Examples

### Temp file creation

In this example we will create three `tmpFile`'s in three different
folders under the `/tmp` directory.


```ruby
sh = Shell.cd("/tmp") # Change to the /tmp directory
sh.mkdir "shell-test-1" unless sh.exists?("shell-test-1")
# make the 'shell-test-1' directory if it doesn't already exist
sh.cd("shell-test-1") # Change to the /tmp/shell-test-1 directory
for dir in ["dir1", "dir3", "dir5"]
  if !sh.exists?(dir)
    sh.mkdir dir # make dir if it doesn't already exist
    sh.cd(dir) do
      # change to the `dir` directory
      f = sh.open("tmpFile", "w") # open a new file in write mode
      f.print "TEST\n"            # write to the file
      f.close                     # close the file handler
    end
    print sh.pwd                  # output the process working directory
  end
end
```

### Temp file creation with self

This example is identical to the first, except we're using
Command`Processor#transact`.

Command`Processor#transact` executes the given block against self, in
this case `sh`; our Shell object. Within the block we can substitute
`sh.cd` to `cd`, because the scope within the block uses `sh` already.


```ruby
sh = Shell.cd("/tmp")
sh.transact do
  mkdir "shell-test-1" unless exists?("shell-test-1")
  cd("shell-test-1")
  for dir in ["dir1", "dir3", "dir5"]
    if !exists?(dir)
      mkdir dir
      cd(dir) do
        f = open("tmpFile", "w")
        f.print "TEST\n"
        f.close
      end
      print pwd
    end
  end
end
```

### Pipe /etc/printcap into a file

In this example we will read the operating system file `/etc/printcap`,
generated by `cupsd`, and then output it to a new file relative to the
`pwd` of `sh`.


```ruby
sh = Shell.new
sh.cat("/etc/printcap") | sh.tee("tee1") > "tee2"
(sh.cat < "/etc/printcap") | sh.tee("tee11") > "tee12"
sh.cat("/etc/printcap") | sh.tee("tee1") >> "tee2"
(sh.cat < "/etc/printcap") | sh.tee("tee11") >> "tee12"
```



## OptionParser

### OptionParser

#### Introduction

OptionParser is a class for command-line option analysis. It is much
more advanced, yet also easier to use, than GetoptLong, and is a more
Ruby-oriented solution.

#### Features

1.  The argument specification and the code to handle it are written in
    the same place.
2.  It can output an option summary; you don't need to maintain this
    string separately.
3.  Optional and mandatory arguments are specified very gracefully.
4.  Arguments can be automatically converted to a specified class.
5.  Arguments can be restricted to a certain set.

All of these features are demonstrated in the examples below. See
`#make_switch` for full documentation.

#### Minimal example


```ruby
require 'optparse'

options = {}
OptionParser.new do |opts|
  opts.banner = "Usage: example.rb [options]"

  opts.on("-v", "--[no-]verbose", "Run verbosely") do |v|
    options[:verbose] = v
  end
end.parse!

p options
p ARGV
```

#### Generating Help

OptionParser can be used to automatically generate help for the commands
you write:


```ruby
require 'optparse'

Options = Struct.new(:name)

class Parser
  def self.parse(options)
    args = Options.new("world")

    opt_parser = OptionParser.new do |opts|
      opts.banner = "Usage: example.rb [options]"

      opts.on("-nNAME", "--name=NAME", "Name to say hello to") do |n|
        args.name = n
      end

      opts.on("-h", "--help", "Prints this help") do
        puts opts
        exit
      end
    end

    opt_parser.parse!(options)
    return args
  end
end
options = Parser.parse %w[--help]

#=>
   # Usage: example.rb [options]
   #     -n, --name=NAME                  Name to say hello to
   #     -h, --help                       Prints this help
```

#### Required Arguments

For options that require an argument, option specification strings may
include an option name in all caps. If an option is used without the
required argument, an exception will be raised. require 'optparse'


```ruby
options = {}
OptionParser.new do |parser|
  parser.on("-r", "--require LIBRARY",
            "Require the LIBRARY before executing your script") do |lib|
    puts "You required #{lib}!"
  end
end.parse!
```

Used:


```ruby
bash-3.2$ ruby optparse-test.rb -r
optparse-test.rb:9:in `<main>`: missing argument: -r (OptionParser::MissingArgument)
bash-3.2$ ruby optparse-test.rb -r my-library
You required my-library!
```

#### Type Coercion

OptionParser supports the ability to coerce command line arguments into
objects for us.

OptionParser comes with a few ready-to-use kinds of type coercion. They
are:

* Date -- Anything accepted by `Date.parse`

* DateTime -- Anything accepted by `DateTime.parse`
* Time -- Anything accepted by `Time.httpdate` or `Time.parse`
* URI -- Anything accepted by `URI.parse`
* Shellwords -- Anything accepted by `Shellwords.shellwords`
* String -- Any non-empty string
* Integer -- Any integer. Will convert octal. (e.g. 124, -3, 040)
* Float -- Any float. (e.g. 10, 3.14, -100E+13)
* Numeric -- Any integer, float, or rational (1, 3.4, 1/3)
* DecimalInteger -- Like `Integer`, but no octal format.
* OctalInteger -- Like `Integer`, but no decimal format.
* DecimalNumeric -- Decimal integer or float.
* TrueClass -- Accepts '+, yes, true, -, no, false' and defaults as
  `true`
* FalseClass -- Same as `TrueClass`, but defaults to `false`
* Array -- Strings separated by ',' (e.g. 1,2,3)
* Regexp -- Regular expressions. Also includes options.

We can also add our own coercions, which we will cover soon.

##### Using Built-in Conversions

As an example, the built-in `Time` conversion is used. The other
built-in conversions behave in the same way. OptionParser will attempt
to parse the argument as a `Time`. If it succeeds, that time will be
passed to the handler block. Otherwise, an exception will be raised.


```ruby
require 'optparse'
require 'optparse/time'
OptionParser.new do |parser|
  parser.on("-t", "--time [TIME]", Time, "Begin execution at given time") do |time|
    p time
  end
end.parse!
```

Used:


```ruby
bash-3.2$ ruby optparse-test.rb  -t nonsense
... invalid argument: -t nonsense (OptionParser::InvalidArgument)
from ... time.rb:5:in `block in <top (required)>`
from optparse-test.rb:31:in `<main>`
bash-3.2$ ruby optparse-test.rb  -t 10-11-12
2010-11-12 00:00:00 -0500
bash-3.2$ ruby optparse-test.rb  -t 9:30
2014-08-13 09:30:00 -0400
```

##### Creating Custom Conversions

The `accept` method on OptionParser may be used to create converters. It
specifies which conversion block to call whenever a class is specified.
The example below uses it to fetch a `User` object before the `on`
handler receives it.


```ruby
require 'optparse'

User = Struct.new(:id, :name)

def find_user id
  not_found = ->{ raise "No User Found for id #{id}" }
  [ User.new(1, "Sam"),
    User.new(2, "Gandalf") ].find(not_found) do |u|
    u.id == id
  end
end

op = OptionParser.new
op.accept(User) do |user_id|
  find_user user_id.to_i
end

op.on("--user ID", User) do |user|
  puts user
end

op.parse!
```

output: bash-3.2$ ruby optparse-test.rb --user 1 #<struct User id=1,
name="Sam"> bash-3.2$ ruby optparse-test.rb --user 2 #<struct User id=2,
name="Gandalf"> bash-3.2$ ruby optparse-test.rb --user 3
optparse-test.rb:15:in `block in find_user`\: No User Found for id 3
(RuntimeError)

#### Complete example

The following example is a complete Ruby program. You can run it and see
the effect of specifying various options. This is probably the best way
to learn the features of `optparse`.


```ruby
require 'optparse'
require 'optparse/time'
require 'ostruct'
require 'pp'

class OptparseExample
  Version = '1.0.0'

  CODES = %w[iso-2022-jp shift_jis euc-jp utf8 binary]
  CODE_ALIASES = { "jis" => "iso-2022-jp", "sjis" => "shift_jis" }

  class ScriptOptions
    attr_accessor :library, :inplace, :encoding, :transfer_type,
                  :verbose, :extension, :delay, :time, :record_separator,
                  :list

    def initialize
      self.library = []
      self.inplace = false
      self.encoding = "utf8"
      self.transfer_type = :auto
      self.verbose = false
    end

    def define_options(parser)
      parser.banner = "Usage: example.rb [options]"
      parser.separator ""
      parser.separator "Specific options:"

      # add additional options
      perform_inplace_option(parser)
      delay_execution_option(parser)
      execute_at_time_option(parser)
      specify_record_separator_option(parser)
      list_example_option(parser)
      specify_encoding_option(parser)
      optional_option_argument_with_keyword_completion_option(parser)
      boolean_verbose_option(parser)

      parser.separator ""
      parser.separator "Common options:"
      # No argument, shows at tail.  This will print an options summary.
      # Try it and see!
      parser.on_tail("-h", "--help", "Show this message") do
        puts parser
        exit
      end
      # Another typical switch to print the version.
      parser.on_tail("--version", "Show version") do
        puts Version
        exit
      end
    end

    def perform_inplace_option(parser)
      # Specifies an optional option argument
      parser.on("-i", "--inplace [EXTENSION]",
                "Edit ARGV files in place",
                "(make backup if EXTENSION supplied)") do |ext|
        self.inplace = true
        self.extension = ext || ''
        self.extension.sub!(/\A\.?(?=.)/, ".")  # Ensure extension begins with dot.
      end
    end

    def delay_execution_option(parser)
      # Cast 'delay' argument to a Float.
      parser.on("--delay N", Float, "Delay N seconds before executing") do |n|
        self.delay = n
      end
    end

    def execute_at_time_option(parser)
      # Cast 'time' argument to a Time object.
      parser.on("-t", "--time [TIME]", Time, "Begin execution at given time") do |time|
        self.time = time
      end
    end

    def specify_record_separator_option(parser)
      # Cast to octal integer.
      parser.on("-F", "--irs [OCTAL]", OptionParser::OctalInteger,
                "Specify record separator (default \\0)") do |rs|
        self.record_separator = rs
      end
    end

    def list_example_option(parser)
      # List of arguments.
      parser.on("--list x,y,z", Array, "Example 'list' of arguments") do |list|
        self.list = list
      end
    end

    def specify_encoding_option(parser)
      # Keyword completion.  We are specifying a specific set of arguments (CODES
      # and CODE_ALIASES - notice the latter is a Hash), and the user may provide
      # the shortest unambiguous text.
      code_list = (CODE_ALIASES.keys + CODES).join(', ')
      parser.on("--code CODE", CODES, CODE_ALIASES, "Select encoding",
                "(#{code_list})") do |encoding|
        self.encoding = encoding
      end
    end

    def optional_option_argument_with_keyword_completion_option(parser)
      # Optional '--type' option argument with keyword completion.
      parser.on("--type [TYPE]", [:text, :binary, :auto],
                "Select transfer type (text, binary, auto)") do |t|
        self.transfer_type = t
      end
    end

    def boolean_verbose_option(parser)
      # Boolean switch.
      parser.on("-v", "--[no-]verbose", "Run verbosely") do |v|
        self.verbose = v
      end
    end
  end

  #
  # Return a structure describing the options.
  #
  def parse(args)
    # The options specified on the command line will be collected in
    # *options*.

    @options = ScriptOptions.new
    @args = OptionParser.new do |parser|
      @options.define_options(parser)
      parser.parse!(args)
    end
    @options
  end

  attr_reader :parser, :options
end  # class OptparseExample

example = OptparseExample.new
options = example.parse(ARGV)
pp options # example.options
pp ARGV
```

#### Shell Completion

For modern shells (e.g. bash, zsh, etc.), you can use shell completion
for command line options.

#### Further documentation

The above examples should be enough to learn how to use this class. If
you have any questions, file a ticket at http://bugs.ruby-lang.org.



## Etc

The Etc module provides access to information typically stored in files
in the /etc directory on Unix systems.

The information accessible consists of the information found in the
/etc/passwd and /etc/group files, plus information about the system's
temporary directory (/tmp) and configuration directory (/etc).

The Etc module provides a more reliable way to access information about
the logged in user than environment variables such as +$USER+.

### Example:


```ruby
require 'etc'

login = Etc.getlogin
info = Etc.getpwnam(login)
username = info.gecos.split(/,/).first
puts "Hello #{username}, I see your login name is #{login}"
```

Note that the methods provided by this module are not always secure. It
should be used for informational purposes, and not for security.

All operations defined in this module are class methods, so that you can
include the Etc module into your class.



## PTY

Creates and managed pseudo terminals (PTYs). See also
http://en.wikipedia.org/wiki/Pseudo\_terminal

PTY allows you to allocate new terminals using ::open or ::spawn a new
terminal with a specific command.

### Example

In this example we will change the buffering type in the `factor`
command, assuming that factor uses stdio for stdout buffering.

If IO.pipe is used instead of PTY.open, this code deadlocks because
factor's stdout is fully buffered.


```ruby
# start by requiring the standard library PTY
require 'pty'

master, slave = PTY.open
read, write = IO.pipe
pid = spawn("factor", :in=>read, :out=>slave)
read.close     # we dont need the read
slave.close    # or the slave

# pipe "42" to the factor command
write.puts "42"
# output the response from factor
p master.gets #=> "42: 2 3 7\n"

# pipe "144" to factor and print out the response
write.puts "144"
p master.gets #=> "144: 2 2 2 2 3 3\n"
write.close # close the pipe

# The result of read operation when pty slave is closed is platform
# dependent.
ret = begin
        master.gets     # FreeBSD returns nil.
      rescue Errno::EIO # GNU/Linux raises EIO.
        nil
      end
p ret #=> nil
```

### License


```ruby
C) Copyright 1998 by Akinori Ito.

This software may be redistributed freely for this purpose, in full
or in part, provided that this entire copyright notice is included
on any copies of this software and applications and derivations thereof.

This software is provided on an "as is" basis, without warranty of any
kind, either expressed or implied, as to any matter including, but not
limited to warranty of fitness of purpose, or merchantability, or
results obtained from use of this software.
```



## Fcntl

Fcntl loads the constants defined in the system's <fcntl.h> C header
file, and used with both the fcntl(2) and open(2) POSIX system
calls.</fcntl.h>

To perform a fcntl(2) operation, use IO::fcntl.

To perform an open(2) operation, use IO::sysopen.

The set of operations and constants available depends upon specific
operating system. Some values listed below may not be supported on your
system.

See your fcntl(2) man page for complete details.

Open /tmp/tempfile as a write-only file that is created if it doesn't
exist:


```ruby
require 'fcntl'

fd = IO.sysopen('/tmp/tempfile',
                Fcntl::O_WRONLY | Fcntl::O_EXCL | Fcntl::O_CREAT)
f = IO.open(fd)
f.syswrite("TEMP DATA")
f.close
```

Get the flags on file `s`\:


```ruby
m = s.fcntl(Fcntl::F_GETFL, 0)
```

Set the non-blocking flag on `f` in addition to the existing flags in
`m`.


```ruby
f.fcntl(Fcntl::F_SETFL, Fcntl::O_NONBLOCK|m)
```



## Readline

The Readline module provides interface for GNU Readline. This module
defines a number of methods to facilitate completion and accesses input
history from the Ruby interpreter. This module supported Edit
Line(libedit) too. libedit is compatible with GNU Readline.

* GNU Readline: http://www.gnu.org/directory/readline.html

* libedit: http://www.thrysoee.dk/editline/

Reads one inputted line with line edit by Readline.readline method. At
this time, the facilitatation completion and the key bind like Emacs can
be operated like GNU Readline.


```ruby
require "readline"
while buf = Readline.readline("> ", true)
  p buf
end
```

The content that the user input can be recorded to the history. The
history can be accessed by Readline::HISTORY constant.


```ruby
require "readline"
while buf = Readline.readline("> ", true)
  p Readline::HISTORY.to_a
  print("-> ", buf, "\n")
end
```

Documented by Kouji Takao <kouji dot="" takao="" at="" gmail=""
com="">.</kouji>



## WIN32OLE

`WIN32OLE` objects represent OLE Automation object in Ruby.

By using WIN32OLE, you can access OLE server like VBScript.

Here is sample script.


```ruby
require 'win32ole'

excel = WIN32OLE.new('Excel.Application')
excel.visible = true
workbook = excel.Workbooks.Add();
worksheet = workbook.Worksheets(1);
worksheet.Range("A1:D1").value = ["North","South","East","West"];
worksheet.Range("A2:B2").value = [5.2, 10];
worksheet.Range("C2").value = 8;
worksheet.Range("D2").value = 20;

range = worksheet.Range("A1:D2");
range.select
chart = workbook.Charts.Add;

workbook.saved = true;

excel.ActiveWorkbook.Close(0);
excel.Quit();
```

Unfortunately, Win32OLE doesn't support the argument passed by reference
directly. Instead, Win32OLE provides WIN32OLE::ARGV or WIN32OLE\_VARIANT
object. If you want to get the result value of argument passed by
reference, you can use WIN32OLE::ARGV or WIN32OLE\_VARIANT.


```ruby
oleobj.method(arg1, arg2, refargv3)
puts WIN32OLE::ARGV[2]   # the value of refargv3 after called oleobj.method
```

or


```ruby
refargv3 = WIN32OLE_VARIANT.new(XXX,
            WIN32OLE::VARIANT::VT_BYREF|WIN32OLE::VARIANT::VT_XXX)
oleobj.method(arg1, arg2, refargv3)
p refargv3.value # the value of refargv3 after called oleobj.method.
```



## Syslog

The syslog package provides a Ruby interface to the POSIX system logging
facility.

Syslog messages are typically passed to a central logging daemon. The
daemon may filter them; route them into different files (usually found
under /var/log); place them in SQL databases; forward them to
centralized logging servers via TCP or UDP; or even alert the system
administrator via email, pager or text message.

Unlike application-level logging via Logger or Log4r, syslog is designed
to allow secure tamper-proof logging.

The syslog protocol is standardized in RFC 5424.



## Open3

Open3 grants you access to stdin, stdout, stderr and a thread to wait
for the child process when running another program. You can specify
various attributes, redirections, current directory, etc., of the
program in the same way as for Process.spawn.

* Open3.popen3 : pipes for stdin, stdout, stderr

* Open3.popen2 : pipes for stdin, stdout
* Open3.popen2e : pipes for stdin, merged stdout and stderr
* Open3.capture3 : give a string for stdin; get strings for stdout,
  stderr
* Open3.capture2 : give a string for stdin; get a string for stdout
* Open3.capture2e : give a string for stdin; get a string for merged
  stdout and stderr

* Open3.pipeline\_rw : pipes for first stdin and last stdout of a
  pipeline
* Open3.pipeline\_r : pipe for last stdout of a pipeline
* Open3.pipeline\_w : pipe for first stdin of a pipeline
* Open3.pipeline\_start : run a pipeline without waiting
* Open3.pipeline : run a pipeline and wait for its completion
