# 21/11/2025

## REPL

# 1. Welcome to Build a Pokedex
We're going to build a Pokedex in a command-line REPL. We'll use the incredible PokéAPI to fetch all of the data we'll need using GET requests. If you're not familiar with Pokemon, or a Pokedex, that's okay! A Pokedex is just a make-believe device that lets us look up information about Pokemon - things like their name, type, and stats.

Learning Goals
Learn how to parse JSON in Go
Practice making HTTP requests in Go
Learn how to build a CLI tool that makes interacting with a back-end server easier
Get hands-on practice with local Go development and tooling
Learn about caching and how to use it to improve performance
Assignment
Before we dive into the project, let's make sure you have everything you'll need on your machine. This is a simple Golang project, so all you'll need is the latest Go toolchain.

Run and submit the CLI tests once Go is installed locally.

# 2. Build

By the end of this step, you'll have a simple working program in Go!

Assignment
Create a main.go file. It should be part of package main in the root of your project and have a main() function that just prints the text "Hello, World!".
Create a Go module in the root of your project. Here's the command:
go mod init MODULE_NAME

I recommend naming the module by its remote Git location (you should store all your projects in Git!). For example, my GitHub name is wagslane so my module name might be github.com/wagslane/pokedexcli.

Build your program:
go build

The executable will be named after the directory containing the main.go file, hence mine was pokedexcli. You should .gitignore the executable.
Run your program:
./pokedexcli

It should print "Hello, World!" to the console!

Run and submit the CLI tests from the root of the repo.

# 3. Tests

By now, you should be familiar enough with unit testing. Like anything, the more you do it, the better you will be at it. More importantly, you will begin to understand how to write code that is easier to test.

We'll use TDD for this part, and Go's testing package.

If you're unsure how to write unit tests in Go, I highly recommend reading Dave Cheney's excellent blog post on the subject.
Assignment
Create a new cleanInput(text string) []string function. For now it should just return an empty slice of strings.
The purpose of this function will be to split the user's input into "words" based on whitespace. It should also lowercase the input and trim any leading or trailing whitespace. For example:

hello world -> ["hello", "world"]
Charmander Bulbasaur PIKACHU -> ["charmander", "bulbasaur", "pikachu"]
Create a new file for some unit tests. I called mine repl_test.go since I put cleanInput in a new file, repl.go (but you can organize your project your way, the only requirement is that the test file ends in_test.go). Create a test suite for the cleanInput function. Here is the basic structure of the test file:
All tests go inside TestXXX functions that take a *testing.T argument:

func TestCleanInput(t *testing.T) {
    // ...
}

Remember to import the testing package if it isn't imported already.
I like to start by creating a slice of test case structs, in this case:

cases := []struct {
 input    string
 expected []string
}{
 {
  input:    "  hello  world  ",
  expected: []string{"hello", "world"},
 },
 // add more cases here
}

Then I loop over the cases and run the tests:

for _, c := range cases {
 actual := cleanInput(c.input)
 // Check the length of the actual slice against the expected slice
 // if they don't match, use t.Errorf to print an error message
 // and fail the test
 for i := range actual {
  word := actual[i]
  expectedWord := c.expected[i]
  // Check each word in the slice
  // if they don't match, use t.Errorf to print an error message
  // and fail the test
 }
}

Once you have at least a few tests, run the tests using go test ./... from the root of the repo. We expect them to fail.
Implement the cleanInput function to make the tests pass.
Run and submit the CLI tests.

# 4. REPL

REPL stands for "read-eval-print loop". It's a common type of program that allows for interactivity. You type in a command, and the program evaluates it and prints the result. You can then type in another command, and so on.

Your native terminal's command line is a REPL! Our Pokedex will also be a REPL. In this step, we're just going to build the bones of the REPL: a loop that parses and cleans an input and prints the first word back to the user. Here is what your program should be able to do after this step:

> go run .
Pokedex > help me
help
Pokedex > Exit
exit
Pokedex > WHAT even is a pokeman?
what

Pokedex > is our own custom command line prompt. The program waited and recorded my input (in the first instance, help me) and didn't continue until I pressed enter.

Assignment
Remove your "Hello, World!" logic.
Create support for a simple REPL
Wait for user input using bufio.NewScanner (this blocks the code and waits for input, once the user types something and presses enter, the code continues and the input is available in the returned bufio.Scanner)
Start an infinite for loop. This loop will execute once for every command the user types in (we don't want to exit the program after just one command)
Use fmt.Print to print the prompt Pokedex > without a newline character
Use the scanner's .Scan and .Text methods to get the user's input as a string
Clean the user's input string
Capture the first "word" of the input and use it to print: Your command was: <first word>
Test your program. Here's my example session:
wagslane@MacBook-Pro-2 pokedexcli % go run .
Pokedex > well hello there
Your command was: well
Pokedex > Hello there
Your command was: hello
Pokedex > POKEMON was underrated
Your command was: pokemon

You can terminate the program by pressing ctrl+c.

Run the CLI again and tee the output (copies the stdout) to a new file called repl.log (and .gitignore the log).
go run . | tee repl.log

Use this as the first input: CHARMANDER is better than bulbasaur.
Use this as the second input: Pikachu is kinda mean to ash.
Terminate the program by pressing ctrl+c.

# 5. Help and Exit

A REPL is only useful if it does something! Our REPL will work using the concept of "commands". A command is a single word that maps to an action.

We're going to support two commands in this step:

help: prints a help message describing how to use the REPL
exit: exits the program
Assignment
Remove your logic that prints the first word (the command) back to the user
Add a callback for the exit command. Commands in our REPL are just callback functions with no arguments, but that return an error. For example:
func commandExit() error

This function should print Closing the Pokedex... Goodbye! then immediately exit the program. I used os.Exit(0).

Create a "registry" of commands. This will give us a nice abstraction for managing the many commands we'll be adding. I created a struct type that describes a command:
type cliCommand struct {
 name        string
 description string
 callback    func() error
}

Then I created a map of supported commands:

map[string]cliCommand{
    "exit": {
        name:        "exit",
        description: "Exit the Pokedex",
        callback:    commandExit,
    },
}

Register the exit command. Update your REPL loop to use the "command" the user typed in to look up the callback function in the registry. If the command is found, call the callback (and print any errors that are returned). If there isn't a handler, just print Unknown command.
Test your program (obviously).
Add a help command, its callback, and register it. It should print:
Welcome to the Pokedex!
Usage:

help: Displays a help message
exit: Exit the Pokedex

You can dynamically generate the "usage" section by iterating over my registry of commands. That way the help command will always be up-to-date with the available commands.
Test your code again manually.

## PokeApi

1. PokeAPI
Now we're going to use the PokeAPI to get some real data from the Pokemon world!

Assignment
Add the map command. It displays the names of 20 location areas in the Pokemon world. Each subsequent call to map should display the next 20 locations, and so on. This will be how we explore the Pokemon world. Example usage:
Pokedex > map
canalave-city-area
eterna-city-area
pastoria-city-area
sunyshore-city-area
sinnoh-pokemon-league-area
oreburgh-mine-1f
oreburgh-mine-b1f
valley-windworks-area
eterna-forest-area
fuego-ironworks-area
mt-coronet-1f-route-207
mt-coronet-2f
mt-coronet-3f
mt-coronet-exterior-snowfall
mt-coronet-exterior-blizzard
mt-coronet-4f
mt-coronet-4f-small-room
mt-coronet-5f
mt-coronet-6f
mt-coronet-1f-from-exterior

Here are some pointers for implementing this command:

You'll need to use the PokeAPI location-area endpoint to get the location areas. Note that this is a different endpoint than the "location" endpoint. Calling the endpoint without an id will return a batch of location areas.
Update all commands (e.g. help, exit, map) to now accept a pointer to a "config" struct as a parameter. This struct will contain the Next and Previous URLs that you'll need to paginate through location areas.
Here's an example of how to make a GET request in Go.
Here's how to unmarshal a slice of bytes into a Go struct.
You can make GET requests in your browser or by using curl! It's convenient for testing and debugging.
Add the mapb (map back) command. It's similar to the map command, however, instead of displaying the next 20 locations, it displays the previous 20 locations. It's a way to go back.
If you're on the first "page" of results, this command should just print "you're on the first page". Example usage:

Pokedex > map
canalave-city-area
eterna-city-area
pastoria-city-area
sunyshore-city-area
sinnoh-pokemon-league-area
oreburgh-mine-1f
oreburgh-mine-b1f
valley-windworks-area
eterna-forest-area
fuego-ironworks-area
mt-coronet-1f-route-207
mt-coronet-2f
mt-coronet-3f
mt-coronet-exterior-snowfall
mt-coronet-exterior-blizzard
mt-coronet-4f
mt-coronet-4f-small-room
mt-coronet-5f
mt-coronet-6f
mt-coronet-1f-from-exterior
Pokedex > map
mt-coronet-1f-route-216
mt-coronet-1f-route-211
mt-coronet-b1f
great-marsh-area-1
great-marsh-area-2
great-marsh-area-3
great-marsh-area-4
great-marsh-area-5
great-marsh-area-6
solaceon-ruins-2f
solaceon-ruins-1f
solaceon-ruins-b1f-a
solaceon-ruins-b1f-b
solaceon-ruins-b1f-c
solaceon-ruins-b2f-a
solaceon-ruins-b2f-b
solaceon-ruins-b2f-c
solaceon-ruins-b3f-a
solaceon-ruins-b3f-b
solaceon-ruins-b3f-c
Pokedex > mapb
canalave-city-area
eterna-city-area
pastoria-city-area
sunyshore-city-area
sinnoh-pokemon-league-area
oreburgh-mine-1f
oreburgh-mine-b1f
valley-windworks-area
eterna-forest-area
fuego-ironworks-area
mt-coronet-1f-route-207
mt-coronet-2f
mt-coronet-3f
mt-coronet-exterior-snowfall
mt-coronet-exterior-blizzard
mt-coronet-4f
mt-coronet-4f-small-room
mt-coronet-5f
mt-coronet-6f
mt-coronet-1f-from-exterior

Run and submit the CLI tests from the root of the repo.

Tips
JSON lint is a useful tool for debugging JSON, it makes it easier to read.
JSON to Go a useful tool for converting JSON to Go structs. You can use it to generate the structs you'll need to parse the PokeAPI response. Keep in mind it sometimes can't know the exact type of a field that you want, because there are multiple valid options. For nullable strings, use *string.
I recommend creating an internal package that manages your PokeAPI interactions. It's not required, but it's a good organizational and architectural pattern.
