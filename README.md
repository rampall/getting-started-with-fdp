# Getting Started with FDP - `fdp-storage` + `fdp-play`

An interactive how-to guide to building dApps on Fair Data Protocol using `fdp-storage` and `fdp-play`.

## Local Development Setup

### Prerequisites:

**Requirements:** Docker must be already installed

#### Install `fdp-play` globally
```shell
$ npm install -g @fairdatasociety/fdp-play
```

#### Spin up a local FDP development environment with fairos
```shell
$ fdp-play start --fairos
```

### Setup & Installation:

#### Clone the repo
```shell
$ git clone https://github.com/rampall/getting-started-with-fdp.git
```

```shell
$ cd getting-started-with-fdp/
```

### Install dependencies
```shell
$ pnpm i
```

### Run application
**Requirements:** `fdp-play` must be running already! 

```shell
$ pnpm dev
```

The application should be running at - http://localhost:5173/

![image](https://user-images.githubusercontent.com/520570/206979944-11a4e3b1-1f29-44d6-8fba-5bf2d5b93f47.png)
