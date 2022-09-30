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
- Software Engineer at InfluxData
- 40+ releases of Telegraf

![bg fit right:44%](img/mermaid.png)

<!-- This is a presenter note for this page. -->
<!-- EXAMPLE: An EXAMPLE directive is not defined in Marp/Marpit, so this works as presenter notes. -->

## What is Telegraf?

Collect data, organizes it, and push it where you want

![right width:350px](img/tea_sipping_tiger.png)

- Written in Go
- Open Source, MIT License
- 12k Github stars

![width:180px](img/qr-code.png)

## Important aspects

![right width:800px](img/marlin.png)

- Single Binary
- Cross Platform
- Github Release
- Dockerfile

## Release Cycle

![right width:270px](img/trikebike.png)

### Maintenance releases

- Every 3 weeks
- bug/security fixes, dependency updates

### Feature releases

- End of every quarter (March, June, September, December)
- Contains new features + fixes and updates

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

## Preparing the code

![width:400px](img/swimming.png)

<!--
_class: lead invert
-->
