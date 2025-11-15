<div align="center">
	<h1>Go SSH Client.</h1>
	<a href="https://github.com/qba73/goph">
		<img src="https://github.com/qba73/goph/raw/main/.github/goph.png" width="200">
	</a>
	<h4 align="center">
		Fast and easy Go ssh client module.
	</h4>
	<p>Goph is a lightweight Go SSH client focusing on simplicity!</p>
</div>

<p align="center">
	<a href="#-installation-and-documentation">Installation</a> ❘
	<a href="#-features">Features</a> ❘
	<a href="#-usage">Usage</a> ❘
	<a href="#-examples">Examples</a> ❘
	<a href="#-license">License</a>
</p>

## Introduction

This project is a detached fork of the [goph](https://github.com/melbahja/goph) originally developed by [melbahja](https://github.com/melbahja). The original project is not maintained. The goal of this version is to address [opened issues](https://github.com/melbahja/goph/issues), [PRs](https://github.com/melbahja/goph/pulls), and add new functionality required to talk with routers and switches from various vendors.

## Installation and Documentation

```bash
go get github.com/qba73/goph
```

## Features

- Easy to use and **simple API**.
- Supports **known hosts** by default.
- Supports connections with **passwords**.
- Supports connections with **private keys**.
- Supports connections with **protected private keys** with passphrase.
- Supports **upload** files from local to remote.
- Supports **download** files from remote to local.
- Supports connections with **ssh agent** (Unix systems only).
- Supports adding new hosts to **known_hosts file**.
- Supports **file system operations** like: `Open, Create, Chmod...`
- Supports **context.Context** for command cancellation.

## Usage

Run a command via ssh:
```go
package main

import (
	"log"
	"fmt"
	"github.com/qba73/goph"
)

func main() {

	// Start new ssh connection with private key.
	auth, err := goph.Key("/home/user/.ssh/id_rsa", "")
	if err != nil {
		log.Fatal(err)
	}

	client, err := goph.New("root", "192.1.1.3", auth)
	if err != nil {
		log.Fatal(err)
	}

	// Defer closing the network connection.
	defer client.Close()

	// Execute your command.
	out, err := client.Run("ls /tmp/")

	if err != nil {
		log.Fatal(err)
	}

	// Get your output as []byte.
	fmt.Println(string(out))
}
```

#### Start Connection With Protected Private Key:
```go
auth, err := goph.Key("/home/user/.ssh/id_rsa", "you_passphrase_here")
if err != nil {
	// handle error
}

client, err := goph.New("root", "192.1.1.3", auth)
```

#### Start Connection With Password:
```go
client, err := goph.New("root", "192.1.1.3", goph.Password("you_password_here"))
```

#### Start Connection With SSH Agent (Unix systems only):
```go
auth, err := goph.UseAgent()
if err != nil {
	// handle error
}

client, err := goph.New("root", "192.1.1.3", auth)
```

#### Upload Local File to Remote:
```go
err := client.Upload("/path/to/local/file", "/path/to/remote/file")
```

#### Download Remote File to Local:
```go
err := client.Download("/path/to/remote/file", "/path/to/local/file")
```

#### Execute Bash Commands:
```go
out, err := client.Run("bash -c 'printenv'")
```

#### Execute Bash Command with timeout:
```go
context, cancel := context.WithTimeout(ctx, time.Second)
defer cancel()
// will send SIGINT and return error after 1 second
out, err := client.RunContext(ctx, "sleep 5")
```

#### Execute Bash Command With Env Variables:
```go
out, err := client.Run(`env MYVAR="MY VALUE" bash -c 'echo $MYVAR;'`)
```

#### Using Goph Cmd:

`Goph.Cmd` struct is like the Go standard `os/exec.Cmd`.

```go
// Get new `Goph.Cmd`
cmd, err := client.Command("ls", "-alh", "/tmp")

// or with context:
// cmd, err := client.CommandContext(ctx, "ls", "-alh", "/tmp")

if err != nil {
	// handle the error!
}

// You can set env vars, but the server must be configured to `AcceptEnv line`.
cmd.Env = []string{"MY_VAR=MYVALUE"}

// Run you command.
err = cmd.Run()
```

🗒️ Just like `os/exec.Cmd` you can run `CombinedOutput, Output, Start, Wait`, and [`ssh.Session`](https://pkg.go.dev/golang.org/x/crypto/ssh#Session) methods like `Signal`...

#### 📂 File System Operations Via SFTP:

You can easily get a [SFTP](https://github.com/pkg/sftp) client from Goph client:
```go

sftp, err := client.NewSftp()

if err != nil {
	// handle the error!
}

file, err := sftp.Create("/tmp/remote_file")

file.Write([]byte(`Hello world`))
file.Close()

```
For more file operations see [SFTP Docs](https://github.com/pkg/sftp).


## Examples

See [Examples](https://github.com/qba73/ssh/blob/master/examples).

## Missing a Feature?

Feel free to open a new issue, or contact me.

## License

Goph is provided under the [MIT License](https://github.com/qba73/goph/blob/master/LICENSE).
