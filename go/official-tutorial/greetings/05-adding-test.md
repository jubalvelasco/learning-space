# Adding Test

1. Create a file whose name ends with `_test.go` (e.g. `greetings_test.go`), this tells the `go test` command that this file contains tests (these files are excluded from normal builds).

2. Write test functions named `TestXxx` that take a `*testing.T` parameter:

    ```go
    package greetings

    import "testing"

    func TestHelloEmpty(t *testing.T) {
        msg, err := Hello("")
        if msg != "" || err == nil {
            t.Errorf(`Hello("") = %q, %v, want "", error`, msg, err)
        }
    }
    ```

3. Run the tests with `go test` (add `-v` for verbose output).