# Error Handling

Import the `errors` package from the standard library

```go
import "errors"
```

Add error handling to a function

**Old Code:**

```go
func Hello(name string) string {
	message := fmt.Sprintf("Hi, %v. Welcome!", name)
	return message
}
```

**New Code:**

```go
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return "", errors.New("empty name")
    }

    // If a name was received, return a value that embeds the name
    // in a greeting message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message, nil
}
```

The caller now has to handle both return values:

```go
message, err := greetings.Hello("")
if err != nil {
    log.Fatal(err)
}
fmt.Println(message)
```