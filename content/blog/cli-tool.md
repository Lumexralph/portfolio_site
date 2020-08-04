---
title: "Building a multipurpose CLI tool with Cobra and Go"
date: 2020-02-14
image: "images/blog/cli.jpg"
description: "Building a multipurpose CLI tool"
author: "Olumide Ogundele"
type: "post"
---

I have always loved how Unix/Linux tools work when using them in the terminal. I had reasons to build a CLI tool with Python which I used to start up any project with Git initialized some time ago [ws](https://github.com/Lumexralph/wset), it doesn't do much. Recently, I got the inspiration to build a multipurpose unit-converter CLI tool taking the inspiration from Google's unit converter, I had to move away from my terminal often to do some conversions and I thought if I had a tool that will avoid leaving the terminal and maintain my focus, this led to creating `uconv`.

>A command-line interface (CLI) processes commands to a computer program in the form of lines of text. The program which handles the interface is called a command-line interpreter or command-line processor. Operating systems implement a command-line interface in a shell for interactive access to operating system functions or services. Such access was primarily provided to users by computer terminals starting in the mid-1960s, and continued to be used throughout the 1970s and 1980s till today on Windows, Unix systems and personal computer systems.

Source [Wikipedia](https://en.wikipedia.org/wiki/Command-line_interface)

Uconv is also a CLI tool written with [Go](http://golang.org/) and [Cobra](https://github.com/spf13/cobra), the full repository can be found [here](https://github.com/Lumexralph/uconv).

### Design of uconv

I want to be able to create a user experience of uconv by doing the following in the terminal
```bash
    uconv temperature 100 --from=c --to=k
```

and get the following output

```bash
    temperature: 30°C ==> 303K`
```

The anatomy of the command is this

>[command] [subcommand] [argument] --flags


### Getting Started

Install Go from [here](https://golang.org/)

create a directory called whatever name you want, I'll use - uconv and open it with your code editor or IDE, I'll use [VS Code](https://code.visualstudio.com/)

```bash
    mkdir uconv && cd uconv && code .
```

Initialise go modules for ease of package or dependency management in your project, it will create a `go.mod` and `go.sum` file when you start installing packages.

```bash
    go mod init
```

Install Cobra into the project

```bash
go get -u github.com/spf13/cobra/cobra
```

If you follow the usage guideline of Cobra, you'll find that you can bootstrap your project with it

`cobra init --pkg-name <your project directory>` like `github.com/Lumexralph/uconv`

When the files and directories are created, we can proceed by working on the base command `cmd/root.go` file to look like this

```go
// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
    Use:   "uconv",
    Short: "A multi-purpose unit converter",
    Long: `uconv is a CLI tool that helps you convert a value from one unit to another
    It can be used for temperature, weight, area, length, currency`,
}

// Execute adds all child commands to the root command and sets flags appropriately.
// This is called by main.main(). It only needs to happen once to the rootCmd.
func Execute() {
    if err := rootCmd.Execute(); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
}
```


When you're done, run the command to install the package as an executable in your local environment

```bash
    go install .
```

If all goes fine, you should be able to be able to use "uconv" in the terminal like you do with `ls, pwd, grep` etc... You can try it out, you should get something similar to this output, it might not be exact

```bash
    uconv is a CLI tool that helps you convert a value from one unit to another
        It can be used for temperature, weight, area, length, currency

    Usage:
    uconv [command]

    Available Commands:
    help        Help about any command
    length      Convert length for different units
    temperature Convert temperature for different units

    Flags:
        --config string   config file (default is $HOME/.uconv.yaml)
    -h, --help            help for uconv

    Use "uconv [command] --help" for more information about a command.
```

The final interesting part is creating the temperature sub-command, add another file called whatever name you want, I will use `temperature.go` in the cmd directory.

Create your flags and add it it to the sub-command which in our case is temperature packaged as `tempCmd`

   ```go
     var tempTo, tempFrom string

    // special function that gets executed when the app is being compiled,
    func init() {
        // create your flags --from and --to or -f or -t
        tempCmd.Flags().StringVarP(&tempFrom, "from", "f", "c", "the unit to convert from")
        tempCmd.Flags().StringVarP(&tempTo, "to", "t", "f", "the unit to convert to")

        // add the temperature command to the the root command
        rootCmd.AddCommand(tempCmd)
    }
   ```

We have the tempCmd created below, it follows the same pattern as the `rootCommand.`

```go
    var tempCmd = &cobra.Command{
    Use:   "temperature",
    Short: "Convert temperature for different units",
    Long: `temperature is a sub-command for uconv (unit converter).

    It helps to convert temperature from one unit to another, Kelvin - k,
    Fahrenheit - f and Celsius - c.

    If no flags are specified, it converts Celsius - c to Fahrenheit - f by default.

    Usage:
        uconv temperature 45
        uconv temperature 45 --from f --to c
        uconv temperature 45 -f f -t c
    `,
    Args: cobra.ExactArgs(1),
    Run:  executeTemperatureCmd,
    }

```
Use:   "temperature" is the name of my sub-command to be able to achieve this

```bash
uconv temperature
```

Short: "Convert temperature for different units", is the short description you get when you typed just uconv in the terminal. While Long is the description of what the command does when you type the following

```bash
uconv temperature --h or -h
```

**Args**: `cobra.ExactArgs(1)`, was used because I only wanted just one value or argument provided since I just want to convert one value to another value, and the best part for me is where the logic happens

Run:  executeTemperatureCmd, this will have a function with the required signature by Cobra, it will get run to respond to the temperature sub-command and since function is a first class citizen in Go, it can be passed as value like I did below.

This is the function definition below

```go
// executeTemperatureCmd - handles the temperature commands with respective flags
func executeTemperatureCmd(cmd *cobra.Command, args []string) {
    // convert the string to int
    arg, err := strconv.ParseInt(args[0], 10, 64)
    if err != nil {
        fmt.Println(err)
        return
    }

    switch {
    case tempFrom == "c" && tempTo == "f":
        fmt.Printf("temperature: %d°C ==> %d°F\n", arg, celsiusToFahrenheit(arg))
    case tempFrom == "f" && tempTo == "c":
        fmt.Printf("temperature: %d°F ==> %d°C\n", arg, fahrenheitToCelsius(arg))
    case tempFrom == "c" && tempTo == "k":
        fmt.Printf("temperature: %d°C ==> %dK\n", arg, celsiusToKelvin(arg))
    case tempFrom == "f" && tempTo == "k":
        fmt.Printf("temperature: %d°F ==> %dK\n", arg, fahrenheitToKelvin(arg))
    case tempFrom == "k" && tempTo == "c":
        fmt.Printf("temperature: %dK ==> %d°C\n", arg, kelvinToCelsius(arg))
    case tempFrom == "k" && tempTo == "f":
        fmt.Printf("temperature: %dK ==> %d°F\n", arg, kelvinToFahrenheit(arg))
    case tempFrom == tempTo:
        fmt.Printf("temperature: %d%s\n", arg, tempTo)
    }
}

// celsiusToFahrenheit - converts Celsius to Fahrenheit
func celsiusToFahrenheit(data int64) int64 {
    return (data * 9 / 5) + 32
}

// celsiusToKelvin - converts Celsius to Kelvin
func celsiusToKelvin(data int64) int64 {
    return data + 273
}

// fahrenheitToCelsius - converts Fahrenheit to Celsius
func fahrenheitToCelsius(data int64) int64 {
    return (data - 32) * 5 / 9
}

// fahrenheitToKelvin - converts Fahrenheit to Kelvin
func fahrenheitToKelvin(data int64) int64 {
    return (data-32)*5/9 + 273
}

// kelvinToCelsius - converts Kelvin to Celsius
func kelvinToCelsius(data int64) int64 {
    return data - 273
}

// kelvinToFahrenheit - converts Kelvin to Fahrenheit
func kelvinToFahrenheit(data int64) int64 {
    return (data-273)*9/5 + 32
}
```

If you are able to get this far, you can recreate it for other sub-commands you might have. Please ensure to consult the Cobra's guidelines if you want more to your CLI tool and I will be glad to hear from you if you have any question or feedback, have fun!

Before I go, here is a cool trick I do when i want to start a new project with VSCode, I created an alias using a function like this;

```bash
alias np='function _newp(){ mkdir $1 && cd $1 && code .; };_newp'
```

call the new command line alias like this

```bash
 np uconv
```
