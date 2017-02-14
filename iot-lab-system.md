# IoT Lab System

## Platform

### Installation
The following will be setup for each physical machine:
- All the machines will be pre-installed with node.js version >= 6.5.0 && npm version >= 3.10.0.
- The physical machines will be added to the VSO [VSIoT](https://mseng.visualstudio.com/_admin/_AgentPool?poolId=279&_a=agents) agent pool.

*For the machine IP address, please use ``` ping hostname``` to resolve the IP address.*

### Windows

#### Machine Info
- Full hostname: vsciot-win-test.guest.corp.microsoft.com
- OS: Win10 64 bit
- Username: vsciot/Bugsfor$

#### Setup
- [Node.js](http://wwww.nodejs.org)
- Update npm: npm install -g npm
- Install VSTS agent for windows following the [tutorial](https://github.com/Microsoft/vsts-agent/blob/master/README.md)
    - During setup, should choose run agent as a service for production environment
    - **Choose vsciot as the service account instead of the default network service account**
        - This is important since the gulp and related tools need local user account.
    - If the logon failed is popped up, following the steps below to fix this issue:
        - Open services.msc in command
        - Find VSTS Agent on the service list
        - Click the properties from the service context menu to bring up the setting dialog, and choose the Log On Tab
        - Select the "This account" radio button, re-input the username and password.

### Ubuntu

#### Machine Info
- Full hostname: vsciot-ubuntu.guest.corp.microsoft.com
- OS: Ubuntu
- Username: vsciot/#Bugsfor$

#### Setup
- Node.js: Following [this](https://nodejs.org/en/download/package-manager/) to install the latest node.js version
- Update npm: sudo npm install -g npm
- Install VSTS agent for Ubuntu following the [tutorial](https://github.com/Microsoft/vsts-agent/blob/master/README.md)
- **Disable password prompt for testing environment**
    - Be careful for this change since it's **NOT** vi and can be destructive and corrupt the system
    - On bash run
        ```
        sudo visudo
        ```
    - Add the following line to the end of the file to bypass password prompt for npm:
        ```
        vsciot ALL = NOPASSWD: /usr/bin/npm
        ```

### Mac:

#### Machine Info
- Full hostname: vsc-imac.guest.corp.microsoft.com
- Username: vsciot/#Bugsfor$1

#### Setup
- Node.js: Following [this](https://nodejs.org/en/download/package-manager/) to install the latest node.js version
- Install VSTS agent for OSX following the [tutorial](https://github.com/Microsoft/vsts-agent/blob/master/README.md)

## Relay System

### Components
 - Raspberry Pi host name: rpi3-relay.guest.corp.microsoft.com
 - 5V 16 Channel Relay
 - Separate 5V power adapter for relay
    - Arduino can also be used to provide 5V DC, as illustrated in the figure 1.
 - USB & Wires for connection

### Raspberry Pi Setup

```
$ git clone https://mseng.visualstudio.com/_git/VSIoT
$ cd VSIoT/iot-lab/relay-system
// This package have to be run inside the Raspberry PI board since it depends on the raspbian tooling chains.
$ npm install rpio
$ npm install
$ node app.js
```
### Testing Board Setup

The board's VCC pin will be wired to the relay NC pin.

### API

####  API Document

The [swagger](https://github.com/swagger-api/swagger-node) will use to quickly setup the REST API.

```
$ npm install -g swagger
```
#### Run

In code root folder of the lab system ./iot-lab/api-document

```
$ swagger project start
```

#### API List
|API         | Method | Description                          |
| ---------- | ------ | ------------------------------       |
|/           | GET    | Get the device information           |
|/relay      | GET    | List all the relay status            |
|/relay/{id} | GET    | Get the relay status for id { id}    |
|/relay/{id}/on | POST | Turn on relay {id} switch         |
|/relay/{id}/off | POST | Turn off relay {id} switch       |
|/relay/{id}/toggle | POST | Toggle relay {id} status      |

### Boards
- The IoT boards are connected to the physical machines via the following forms. The specific connection form depends on the type and model of the boards.
    - LAN
    - USB
    - WIFI

## Open Issues

### How to detect whether a relay is working normally?
### How to manage the relay system together with IOT boards?
