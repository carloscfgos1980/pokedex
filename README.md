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

# 2. Caching

It's time to implement caching! This will make moving around the map feel a lot snappier. We'll be building a flexible caching system to help with performance in future steps.

What Is a Cache?
A cache temporarily stores data so that future requests for that data can be served faster.

In our case, we'll be caching responses from the PokeAPI so that when we need that same data again, we can grab it from memory instead of making another network request.

Assignment
Create a new internal package called pokecache in your internal directory (if you haven't already created an internal directory in your project, do so now). This package will be responsible for all of our caching logic.
I used a Cache struct to hold a map[string]cacheEntry and a mutex to protect the map across goroutines. A cacheEntry should be a struct with two fields:

createdAt - A time.Time that represents when the entry was created.
val - A []byte that represents the raw data we're caching.
You'll probably want to expose a NewCache() function that creates a new cache with a configurable interval (time.Duration).

Create a cache.Add() method that adds a new entry to the cache. It should take a key (a string) and a val (a []byte).
Create a cache.Get() method that gets an entry from the cache. It should take a key (a string) and return a []byte and a bool. The bool should be true if the entry was found and false if it wasn't.
Create a cache.reapLoop() method that is called when the cache is created (by the NewCache function). Each time an interval (the time.Duration passed to NewCache) passes it should remove any entries that are older than the interval. This makes sure that the cache doesn't grow too large over time. For example, if the interval is 5 seconds, and an entry was added 7 seconds ago, that entry should be removed.
I used a time.Ticker to make this happen.

Maps are not thread-safe in Go. You should use a sync.Mutex to lock access to the map when you're adding, getting entries or reaping entries. It's unlikely that you'll have issues because reaping only happens every ~5 seconds, but it's still possible, so you should make your cache package safe for concurrent use.

Update your code that makes requests to the PokeAPI to use the cache. If you already have the data for a given URL (which is our cache key) in the cache, you should use that instead of making a new request. Whenever you do make a request, you should add the response to the cache.
Write at least 1 test for your cache package! The tip below should help you get started.
Test your application manually to make sure that the cache works as expected. When you use the map command to get data for the first time there should be a noticeable waiting time. However, when you use mapb it should be instantaneous because the data for that page is already in the cache. Feel free to add some logging that informs you in the command line when the cache is being used.
Run and submit the CLI tests from the root of the repo.

Tip
You can run tests for all packages in a Go module by running go test ./... from the root of the module.

func TestAddGet(t *testing.T) {
 const interval = 5* time.Second
 cases := []struct {
  key string
  val []byte
 }{
  {
   key: "<https://example.com>",
   val: []byte("testdata"),
  },
  {
   key: "<https://example.com/path>",
   val: []byte("moretestdata"),
  },
 }

 for i, c := range cases {
  t.Run(fmt.Sprintf("Test case %v", i), func(t *testing.T) {
   cache := NewCache(interval)
   cache.Add(c.key, c.val)
   val, ok := cache.Get(c.key)
   if !ok {
    t.Errorf("expected to find key")
    return
   }
   if string(val) != string(c.val) {
    t.Errorf("expected to find value")
    return
   }
  })
 }
}

func TestReapLoop(t *testing.T) {
 const baseTime = 5 * time.Millisecond
 const waitTime = baseTime + 5*time.Millisecond
 cache := NewCache(baseTime)
 cache.Add("<https://example.com>", []byte("testdata"))

 _, ok := cache.Get("https://example.com")
 if !ok {
  t.Errorf("expected to find key")
  return
 }

 time.Sleep(waitTime)

 _, ok = cache.Get("https://example.com")
 if ok {
  t.Errorf("expected to not find key")
  return
 }
}

# 3. Explore

After a user uses the map commands to find a location area, we want them to be able to see a list of all the Pokémon located there.

Assignment
Add an explore command. It takes the name of a location area as an argument.

Run and submit the CLI tests.

Tips
Use the same PokeAPI location-area endpoint, but this time you'll need to pass the name of the location area being explored. By adding a name or id, the API will return a lot more information about the location area.
Feel free to use tools like JSON lint and JSON to Go to help you parse the response.
Parse the Pokemon's names from the response and display them to the user.
Make sure to use the caching layer again! Re-exploring an area should be blazingly fast.
You'll need to alter the function signature of all your commands to allow them to allow parameters. E.g. explore <area_name>
Example usage:

Pokedex > explore pastoria-city-area
Exploring pastoria-city-area...
Found Pokemon:

- tentacool
- tentacruel
- magikarp
- gyarados
- remoraid
- octillery
- wingull
- pelipper
- shellos
- gastrodon
Pokedex >

# 4. Catch

It's time to catch some pokemon! Catching Pokemon adds them to the user's Pokedex.

Assignment
Add a catch command. It takes the name of a Pokemon as an argument. Example usage:
Pokedex > catch pikachu
Throwing a Pokeball at pikachu...
pikachu escaped!
Pokedex > catch pikachu
Throwing a Pokeball at pikachu...
pikachu was caught!

Be sure to print the Throwing a Pokeball at <pokemon>... message before determining if the Pokemon was caught or not.
Use the Pokemon endpoint to get information about a Pokemon by name.
Give the user a chance to catch the Pokemon using the math/rand package.
You can use the pokemon's "base experience" to determine the chance of catching it. The higher the base experience, the harder it should be to catch.
Once the Pokemon is caught, add it to the user's Pokedex. I used a map[string]Pokemon to keep track of caught Pokemon.
Test the catch command manually - make sure you can actually catch a Pokemon within a reasonable number of tries.

# 5. Inspect

Just like in the Pokemon game, our Pokedex will only allow players to see details about a Pokemon if they have seen it before (or in our case, caught it)

Assignment
Add an inspect command. It takes the name of a Pokemon and prints the name, height, weight, stats and type(s) of the Pokemon. Example usage:

Pokedex > inspect pidgey
you have not caught that pokemon
Pokedex > catch pidgey
Throwing a Pokeball at pidgey...
pidgey was caught!
Pokedex > inspect pidgey
Name: pidgey
Height: 3
Weight: 18
Stats:
  -hp: 40
  -attack: 45
  -defense: 40
  -special-attack: 35
  -special-defense: 35
  -speed: 56
Types:

- normal
- flying

You should not need to make an API call to get this information, since you should have already stored it when the user caught the Pokemon.

If the user has not caught the Pokemon, just print a message saying so.

## POKEDEX

# 1. Pokedex

The main purpose of a Pokedex is to keep track of all the Pokemon you've caught. Gotta catch 'em all!

Assignment
Add a pokedex command. It takes no arguments but prints a list of all the names of the Pokemon the user has caught. Example usage:

Pokedex > catch pidgey
Throwing a Pokeball at pidgey...
pidgey was caught!
You may now inspect it with the inspect command.
Pokedex > catch caterpie
Throwing a Pokeball at caterpie...
caterpie was caught!
You may now inspect it with the inspect command.
Pokedex > pokedex
Your Pokedex:

- pidgey
- caterpie
