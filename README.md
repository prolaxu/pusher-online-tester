# Pusher Tester

## Overview
This project is a simple web application designed to test the integration of Pusher's real-time communication features. It allows users to connect to Pusher, subscribe to channels, and receive real-time updates. This README provides an overview of the project, setup instructions, and usage guidelines.

## Live Application
You can access the live version of the application here: [Pusher Tester](https://main--pusher-tester.netlify.app/#/)


## Features
- **Real-Time Messaging**: Connects to Pusher and listens for messages on specified channels.
- **Connection State Monitoring**: Displays the current connection state (connected, disconnected, etc.).
- **Event Simulation**: Allows for testing event emission and reception through simulated connections.

## Getting Started

### Prerequisites
- Node.js installed on your machine.
- A Pusher account with API keys.



## Clone the Repository:
```bash
git clone https://github.com/prolaxu/pusher-online-tester.git
cd pusher-online-tester
```


## Install the dependencies
```bash
yarn
# or
npm install
```

### Start the app in development mode (hot-code reloading, error reporting, etc.)
```bash
quasar dev
```


### Lint the files
```bash
yarn lint
# or
npm run lint
```


### Format the files
```bash
yarn format
# or
npm run format
```



### Build the app for production
```bash
quasar build
```

