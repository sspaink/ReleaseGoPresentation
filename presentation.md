---
marp: true
---

<!--
theme: gaia
class: invert
headingDivider: 2 
paginate: true
style: |
    section{
      justify-content: flex-start;
    }
    img[alt~="right"] {
      float: right;
    }
-->

<!--
_class: lead invert
-->

# Shipping Go Without Sinking

![width:600px](img/sinking.png)

Go West Conference 2022

## Speaker Introduction

- **Name:** Sebastian Spaink
- Writing Go for 4 years
- Software Engineer at InfluxData

![bg fit right:44%](img/mermaid.png)

## What is Telegraf?

Collect data, organizes it, and push it where you want

![right width:350px](img/tea_sipping_tiger.png)

- 40+ releases of Telegraf
- Open Source, MIT License
- 1000+ Unique Contributors

![width:180px](img/qr-code.png)

## A successful release of Telegraf

![right width:700px](img/marlin.png)

- Single Binary
- Cross Platform
- Github Release
- Website release
- Dockerfile

## **Release Cycle**

![right width:400px](img/trikebike.png)

### Feature releases

- End of every quarter
(March, June, September, December)

### Maintenance releases

- Every 3 weeks

<!-- deprecated release candidates -->
<!-- every month releases in future -->

## Semantic Versioning

**example current version** 1.19.0

| Type       | Version | Description                           |
| -----------|---------|---------------------------------------|
| **Patch:** | 1.19.1  | maintenance release                   |
| **Minor:** | 1.20.0  | feature release                       |
| **Major:** | 2.0.0   | feature release with breaking changes |

## Starting the Release

- Eeny, meeny, miny moe, Catch a tiger by the toe
- Track progress in slack

![width:600px](img/slack-progress.gif)

## Steps Overview

1. Preparing the code
2. Building and packaging
3. Distributing the binaries

![bg fit right](img/robot.png)

<!--
_class: lead invert
-->

## Preparing the Code

![width:400px](img/swimming.png)

<!--
_class: lead invert
-->

## Conventional Commit Messages

Adding human and machine readable meaning to commit messages

**Example:** feat(inputs.directory_monitor): Traverse sub-directories
| Type | Optional Scope       | Description |
| -----------                 | -----------                  | -----------  |
| feat          | inputs.directory_monitor           | Traverse sub-directories |

## Git branching strategy

![right width:600px](img/watering.png)

- copy master branch
- branch for each release
- maintain release branch

## Executing external commands

```go
func RunCommand(path, command string, arguments ...string) (string, error) {
    var w io.Writer
    var stdBuffer bytes.Buffer
    w = io.MultiWriter(os.Stdout, &stdBuffer)
    
    cmd := exec.Command(command, arguments...)
    cmd.Dir = path
    cmd.Stdout = w
    cmd.Stderr = w
    err := cmd.Run()
    if err != nil {
        return "", err
    }
    return stdBuffer.String(), nil
}
```

## Adding some color

```go
// Wrap color implements the io.Writer interface
// Using fatih/color package it prints to stdout but with color
type WrapColor struct{}

func (w *WrapColor) Write(b []byte) (int, error) {
    color.Cyan(string(b))
    return len(b), nil
}

…………

var w io.Writer
var stdBuffer bytes.Buffer
var c WrapColor
w = io.MultiWriter(&c, &stdBuffer)
```

## Running the commands

```go
RunCommand(telegrafPath, "make", "tidy")
RunCommand(telegrafPath, "make", "test")

RunCommand(path, "git", "add", strings.Join(changes, " "))
RunCommand(path, "git", "commit", "-m", commitMsg)
RunCommand(path, "git", "push", "origin", branch)
hash, _ := RunCommand(path, "git", "rev-parse", "HEAD")
```

## Working with git commits

```go
type Commit struct {
    Hash            string
    AuthorName      string
    Type            string // (e.g. `feat`)
    Scope           string // (e.g. `core`)
    Subject         string // (e.g. `Add new feature`)
    Title           string // (e.g. `feat(core): Add new feature`)
}
```

## Collecting git commits

```go
separator := "@@__CHGLOG__@@"
delimiter := "@@__CHGLOG_DELIMITER__@@"
logFormat := separator + strings.Join([]string{
    "HASH", 
    "AUTHOR",
    "SUBJECT",
    "BODY",
}, delimiter)

arguments := []string{"log", fmt.Sprintf("--pretty=%s", logFormat), fromBranch + ".." + toBranch}
RunCommand(path, "git", arguments...)
```

## Changelog template

```yaml
## {{ .Version }} [{{ .Date }}]
{{ range .CommitGroups }}
### {{ println .Title }}
{{ range .Commits -}}
- {{.PullRequestLink}} {{ if .Scope }}`{{ .Scope }}` {{ end }}{{ println .Subject }}
{{- end -}}
{{ end -}}
```

## **Example**

### v1.24.1 [2022-09-19]

#### Bugfixes

- [#11787](https://github.com/influxdata/telegraf/pull/11787) Clear error message when provided config is not a text file

#### Dependency Updates

- [#11788](https://github.com/influxdata/telegraf/pull/11788) `deps` Bump cloud.google.com/go/pubsub from 1.24.0 to 1.25.1

## **Preparing the code:** lessons learned

- Organizing commits with conventional commit messages
- Write the release tools the team knows best
- Allow tools to be re-run

## **Preparing the code:** future improvements

- Cherry-pick changes into release branch continuously
- Update changelog continuously

## Building and Packaging

![width:600px](img/package.png)

<!--
_class: lead invert
-->

## The Makefile

- make test
- make build
- make package include_packages=”$(make windows)”

```makefile
LDFLAGS := $(LDFLAGS) -X $(INTERNAL_PKG).Commit=$(commit) -X $(INTERNAL_PKG).Branch=$(branch)
ifneq ($(tag),)
    LDFLAGS += -X $(INTERNAL_PKG).Version=$(version)
else
    LDFLAGS += -X $(INTERNAL_PKG).Version=$(version)-$(commit)
endif

.PHONY: build
build:
    go build -tags "$(BUILDTAGS)" -ldflags "$(LDFLAGS)" ./cmd/telegraf
```

## Mage

```go
func build(p platform) (string, error) {
    env := map[string]string{"GOOS": p.OS, "GOARCH": p.ARCH}
    folderName := fmt.Sprintf("%s_%s", p.OS, p.ARCH)
    binName := productName + p.Extension
    filePath := filepath.Join("bin", folderName, binName)
    err := runCmd(env, "go", "build", "-o", filePath, "cmd/main.go")
    if err != nil {
        return "", err
    }

    return filePath, nil
}
```

Check it out: https://github.com/magefile/mage 

##

![bg fit ](img/ci.png)

## Automatic Alpha Builds

![bg 80%](img/tigerbot.png)

## Adding version and icon to Windows

Using package josephspurrier/goversioninfo

```go
//go:generate goversioninfo -icon=../../assets/windows/tiger.ico
```

```json
{
 "StringFileInfo": {
  "ProductName": "Telegraf",
  "ProductVersion": "1.25.0-71b4a0af"
 }
}
```

## Generating versioninfo.json

```go
version, _ := exec.Command("make", "version").Output()

v := VersionInfo{
    StringFileInfo: StringFileInfo{
        ProductName:    "Telegraf",
        ProductVersion: version,
    },
}

if err := ioutil.WriteFile("cmd/telegraf/versioninfo.json", file, 0644); err != nil {
    log.Fatalf("Failed to write versioninfo.json: %v", err)
}
```

##

![bg fit](img/winexe.png)

![bg fit](img/windowsprop.png)

## **Building and Packaging:** lessons learned

- Don’t run the CI on wednesdays
- Make artifacts readily available
- Sign mac nightly, replicate release pipeline

## **Building and Packaging:** future improvements

- Migrate more steps to continuous integration pipeline

## Distributing Binaries

![width:600px](img/GopherAndTiger.png)

<!--
_class: lead invert
-->

## 

![bg fit](img/github.png)

## Updating the Website


```go
b, err = sjson.SetBytes(b, "telegraf_stable.version", version)
if err != nil {
    return nil, fmt.Errorf("failed to set telegraf_stable.version: %w", err)
}
```

##

![bg fit](img/website.png)

## Testing Dockerfile

```go
container, _ := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
    ContainerRequest: req,
    Started:          true,
})

name, _ := container.Name(ctx)

cmd := exec.Command("docker", "exec", name, "telegraf", "--version")
out, _ := cmd.CombinedOutput()

// Verify the correct version of Telegraf exists
if !strings.Contains(string(out), ver.String()) {
    return fmt.Errorf("Expected docker to give telegraf version %s but got %s", version, string(out))
}
```

## **Distributing Binaries:** lessons learned

- Document all permissions required
- If there is a problem increment patch, or the hash will change

## **Distributing Binaries:** future improvements

- Migrate more steps to continuous integration pipeline

## Conquering a Major Release

- Breaking changes
- Backwards compatibility important
- Deprecating configuration options slowly
- Getting into the habit

![bg fit right:50%](img/princess_tower.png)