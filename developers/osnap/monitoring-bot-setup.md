# 🤖 Monitoring Bot Setup

oSnap module resolves all transaction proposals through UMA Optimistic Oracle V3. Any proposal that is not disputed during the liveness period can be executed by anyone unless there is a [transaction guard](https://help.safe.global/en/articles/5496893-add-a-transaction-guard) configured on the Gnosis Safe. Hence, it is critical to monitor the oSnap module in order not to miss any transaction proposals or configuration changes.

This tutorial walks through example on how to setup oSnap monitoring bot from UMA [protocol repository](https://github.com/UMAprotocol/protocol/) that watches for emitted contract events and sends alerts to selected channels (Slack, Discord or PagerDuty).

### Installation

#### Docker method

The instructions assume you have [Docker](https://www.docker.com/) installed and its server daemon is running.

Get the latest UMA protocol image that among others includes the required monitoring bot:

```bash
docker pull umaprotocol/protocol:latest
```

#### Node package method

The instructions assume you have already the latest long term support version of [Node.js](https://nodejs.dev).

Initialize new project directory and install `@uma/monitor-v2` package that contains the oSnap monitoring bot:

```bash
mkdir monitor-osnap
cd monitor-osnap
npm init -y
npm install @uma/monitor-v2
```

### Basic configuration

You can select among multiple supported notification channels where oSnap monitor bot would be sending its alerts:

* Slack: requires webhook URL for the channel. Please see [Slack documentation](https://api.slack.com/messaging/webhooks) on how to obtain it.
* Discord:  requires webhook URL for the channel. Please see [guide](https://hookdeck.com/webhooks/platforms/how-to-get-started-with-discord-webhooks#what-do-webhooks-do-in-discord) on how to obtain it.
* PagerDuty: requires integration key that can be obtained by adding Events API v2 integration to the selected PagerDuty service. Please see [PagerDuty documentation](https://support.pagerduty.com/docs/services-and-integrations#create-a-generic-events-api-integration) for further details.

All the configuration for the monitor bot should be set in the environment variables. Please see below the basic example `.env` file that should be placed in your working directory. Make sure to assign all required values using the guidance in the comments below:

{% code overflow="wrap" %}
```sh
# Note on formatting: Do not enclose variable values in quotes as this is not supported when running the bot in Docker container.

# Address of the oSnap module being monitored. Caution: make sure it is the address of the oSnap module, not the controlled Gnosis Safe.
OG_ADDRESS=

# Numeric chain identifier for the network where the oSnap module is deployed, e.g. for Ethereum mainnet:
CHAIN_ID=1

# RPC node URL for monitoring the network where the oSnap module is deployed. Variable key is in the form NODE_URL_X (replace X with the actual chain identifier), e.g. for Ethereum mainnet:
NODE_URL_1=

# Stringified JSON configuration in the form of {"defaultWebHookUrl":"<SLACK_WEBHOOK>"} where <SLACK_WEBHOOK> should be replaced with the URL of Slack webhook. If configured this will send alerts to the the selected Slack channel.
SLACK_CONFIG=

# Stringified JSON configuration in the form of {"defaultWebHookUrl":"<DISCORD_WEBHOOK>"} where <DISCORD_WEBHOOK> should be replaced with the URL of Discord webhook. If configured this will send alerts to the the selected Discord channel.
DISCORD_CONFIG=

# Stringified JSON configuration in the form of {"integrationKey":"<SERVICE_INTEGRATION_KEY>"} where <SERVICE_INTEGRATION_KEY> should be replaced with the integration key of PagerDuty service. If configured this will send alerts to the the selected PagerDuty service.
PAGER_DUTY_V2_CONFIG=

# Boolean enabling/disabling monitoring of proposed transactions (false by default).
TRANSACTIONS_PROPOSED_ENABLED=true

# Boolean enabling/disabling monitoring of proposed transaction execution (false by default).
PROPOSAL_EXECUTED_ENABLED=true

# Boolean enabling/disabling monitoring of deleted transaction proposals (false by default). Normally proposed transactions are deleted on Optimistic Oracle dispute
PROPOSAL_DELETED_ENABLED=true
```
{% endcode %}

### Running the bot

#### Docker method

Instruct docker to run oSnap monitor bot by appending `COMMAND` environment variable to the same `.env` file where other configuration is stored:

```bash
# Command to run when starting docker container:
COMMAND=node ./packages/monitor-v2/dist/monitor-og/index.js
```

Start the monitoring bot from the same working directory where the `.env` configuration file is located:

```bash
docker run -d --env-file .env --name osnap-monitor --rm umaprotocol/protocol:latest
```

This should start the oSnap module monitoring bot in a looping mode detached from console where all configured events will be sent to the provided notification channels.

Stop the running bot container with:

```bash
docker stop -t 0 osnap-monitor
```

#### Node package method

Start the monitoring bot from the root of your project directory where node package was installed and `.env` configuration file is stored:

```bash
node node_modules/@uma/monitor-v2/dist/monitor-og/index.js
```

This should start the oSnap module monitoring bot in a looping mode where all configured events will be logged both on console and forwarded to the provided notification channels.

### Advanced configuration

For reference on the other available configuration options please see environmental variables that can be set in the example `.env` file in addition to basic configuration covered above:

{% code overflow="wrap" %}
```bash
# Boolean enabling/disabling monitoring of changing proposal bond currency and amount (false by default).
SET_COLLATERAL_BOND_ENABLED=true

# Boolean enabling/disabling monitoring of changing oSnap rules parameter (false by default).
SET_RULES_ENABLED=true

# Boolean enabling/disabling monitoring of changing available liveness period for disputing transaction proposals (false by default).
SET_LIVENESS_ENABLED=true

# Boolean enabling/disabling monitoring of changing DVM request identifier (false by default).
SET_IDENTIFIER_ENABLED=true

# Boolean enabling/disabling monitoring of changing Escalation Manager to be used for asserting proposals at Optimistic Oracle V3 (false by default).
SET_ESCALATION_MANAGER_ENABLED=true

# Name of the bot identifier as shown in alerts.
BOT_IDENTIFIER=oSnap Monitor Bot

# Value in seconds for delay between consecutive runs, defaults to 60 seconds. If set to 0 then this assumes serverless environment where the process exits after the first loop.
POLLING_DELAY=60

# Specific block range to look for events (this is mandatory when POLLING_DELAY=0). Variable keys are in the form STARTING_BLOCK_NUMBER_X and ENDING_BLOCK_NUMBER_X (replace X with the actual chain identifier), e.g. for Ethereum mainnet:
STARTING_BLOCK_NUMBER_1=
ENDING_BLOCK_NUMBER_1=

# Stringified JSON array of RPC node URLs if using multiple providers for redundancy. Variable key is in the form NODE_URLS_X (replace X with the actual chain identifier), e.g. for Ethereum mainnet:
NODE_URLS_1=

# Number of retries to make when a RPC node request fails (defaults to 2).
NODE_RETRIES=2

# Delay in seconds between retries when RPC node request fails (defaults to 1).
NODE_RETRY_DELAY=1

# Timeout in seconds for RPC node requests (defaults to 60).
NODE_TIMEOUT=60
```
{% endcode %}
