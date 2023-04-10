# Lab: Agents Connecting<!-- omit in toc -->

- [Overview](#overview)
- [How to Run](#how-to-run)
- [Instructions](#instructions)
- [Navigating the Code](#navigating-the-code)
  - [Faber Controller](#faber-controller)
    - [Faber Agent Startup](#faber-agent-startup)
    - [Connection Invitation Creation](#connection-invitation-creation)
    - [User input Processing](#user-input-processing)
    - [Send a Basic Message](#send-a-basic-message)
  - [Alice Controller](#alice-controller)
    - [Alice Agent Startup](#alice-agent-startup)
    - [Accept Invitation](#accept-invitation)
    - [User input Processing](#user-input-processing-1)
    - [Send a Basic Message](#send-a-basic-message-1)
- [Takeaways](#takeaways)

## Overview

In this lab, we spin up a couple of Aries agents, establish connections between them and have them exchange messages using the Aries `Basic Message` Protocol.

## How to Run

This lab from the ACA-Py repository includes details about running the example locally using Docker, on Play with Docker, and even has some guidance on running the lab without Docker. As always, we recommend running the lab using Docker, so you don’t get bogged down in technical issues unrelated to the lessons of the lab.

## Instructions

This tutorial in the ACA-Py repository spins up agents for Alice and Faber and has them connect and exchange messages. Follow the [instructions here](https://github.com/hyperledger/aries-cloudagent-python/tree/main/demo#the-alicefaber-python-demo), stopping at the end of the “Exchanging Messages” section. We’ll do the rest of the tutorial later (which admittedly, isn’t much!).

One important thing to understand about this particular demo is the agent and controller integration. As we stressed in the course, ACA-Py runs two processes: one for the agent and one for the controller. However, in this demo it appears that the two are run with one command—and that can be a little confusing. In fact, they don’t run in the same **process**. The demo uses a Python feature such that the controller starts the ACA-Py instance as a sub-process. Use the links below to see the relevant code for that:

- AliceAgent is an instance of demo agent - [code](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/alice.py#L107)
- Demo agent starts a sub-process that is an instance of ACA-Py - [code](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/support/agent.py#L676)

There has been discussion of changing this demo to make the separation of controller and ACA-Py instance more obvious. In a later lab we’ll see examples of other controllers for Alice and Faber using the same ACA-Py instance, making the separation more obvious.

## Navigating the Code

Now let’s take a look at the controller code. We’ll cover here the important parts of the controllers starting up, connecting and sending text messages. In a later lab, we’ll complete the demo through issuing and proving verifiable credentials and cover those parts of the controller code. Since we aren’t dealing with verifiable credentials in this part of the code walk through, little of what is covered here relates to AnonCreds or an Indy ledger.

> **Note:** The links in these instructions to specific lines in the code go to a
specific version (commit) of the files in GitHub that may not be the latest. As
such, the line numbers in GitHub may be slightly different than what is on your
editor (the latest version on the main branch). The links are using [commit
467104fab662507e18483abaaf2305bc702bb303](https://github.com/hyperledger/aries-cloudagent-python/tree/467104fab662507e18483abaaf2305bc702bb303) from February 11, 2023. We'll
be doing an update of these links to a more recent commit in the near future.

- Alice agent code is in the repo file [demo/runners/alice.py](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/alice.py)
- Faber agent code is in the file [demo/runners/faber.py](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/faber.py)
- Both Alice and Faber are instances of agents in the file [demo/runners/agent_container.py](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/agent_container.py)
- The Alice and Faber agents are instances of the DemoAgent class imported into agent container from [demo/runners/support/agent.py](https://github.com/hyperledger/aries-cloudagent-python/tree/467104fab662507e18483abaaf2305bc702bb303/demo/runners/support/agent.py)

### Faber Controller

#### Faber Agent Startup

- The [“main” function](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/faber.py#L375) to start the controller and agent.
  - “agent = FaberAgent …” start an agent instance.
    - Call to the (parent) [controller class](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/faber.py#L41) to initialize the controller.
    - Note the specific details about Faber (e.g. port numbers) and the extra ACA-Py startup parameters just for Faber.
  - “await self.listen_webhooks …” setup for receiving events from ACA-Py.
    - Parent [method](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/support/agent.py#L544) that initializes webhooks and route handlers.
      - Note below the functions for receiving and handling webhooks from ACA-Py. These functions are called when an event happens in ACA-Py that the controller needs to know about.
      - Also just below that are the admin calls (request, GET and POST) for the controller to invoke the ACA-Py endpoints.
  - “await agent.start_process … “ call to startup the ACA-Py instance process.
    - Parent [method](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/agent_container.py#L46) that starts the ACA-Py instance sub-process,
    - Based on the [DemoAgent method](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/support/agent.py#L706)
      - The ACA-Py process is separate from the controller process.
    - “agent_args = …” prepare ACA-Py startup arguments.
      - [Function](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/support/agent.py#L328) that gathers all the ACA-Py arguments to use, defaults, specific ports, URLs, wallet (storage info), optional demo arguments and controller-specific arguments. So many!!!
      - Recall our earlier look at all of the possible ACA-Py startup arguments as you look at how many we are using in this example.
- The [method in the agent class](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/support/agent.py#L116) to initialize the controller instance.
  - Like ACA-Py, many parameters may be passed in to control specific behavior. As seen above, Faber only specifies a handful of those parameters.
  - “self.ident = …“ defaulting of parameters as needed.
  - “if RUN_MODE …” is special handling to support running on “[Play with Docker](https://labs.play-with-docker.com/)”.
  - “self.storage_type = …” specify the type and details of the agent storage to be used.
    - The demo supports (with appropriate configuration) the use of SQLite (default) or PostgreSQL database for agent secure storage.
    - Agent secure storage uses the "askar" ([Aries Askar](https://github.com/hyperledger/aries-askar))  implementation by default, but can use the older indy-sdk ("indy") as well.

#### Connection Invitation Creation

- [Call to generate](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/faber.py#L430) an invitation that Alice can use to connect.
- [Generate](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/agent_container.py#L563) the invitation, perhaps with an on screen QR code.
  - “invi_rec = await …” call to ACA-Py to create an invitation.
  - “await self.detect_connection …” wait a response to the invitation.

#### User input Processing

- [Main loop](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/faber.py#L453) for processing command line input from the user.

#### Send a Basic Message

- The user [chose Option 3](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/faber.py#L679), send a message.
  - Prompt for the message.
  - “await faber_agent.agent.admin_POST …” invoke ACA-Py to start an instance of the Basic Message protocol ([RFC 0095](https://github.com/hyperledger/aries-rfcs/tree/main/features/0095-basic-message)).

### Alice Controller

#### Alice Agent Startup

The [startup of the Alice controller](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/alice.py#L106) is almost the same as Faber, so we’ll leave the review of that part of the code as an exercise for the reader. Much of the differences between the two are in the areas that relate to the handling of verifiable credentials, which we cover in a later exercise.

#### Accept Invitation

- [Prompt for and receive the text invitation](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/alice.py#L65), as pasted in by the user.
  - "try … url = …“ first try at parsing the invitation text.
    - The controller allows flexibility in what the user pastes in. It could be a URL with a base64 invitation as a query parameter, only the base64 query parameter, or the already decoded JSON of the invitation.
    - Or it might be an improperly constructed invitation and so not a valid invitation at all.
  - “connection = await …” invokes the agent_container's [invitation handler](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/agent_container.py#L601), which invokes the base class [AgentDemo handling](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/support/agent.py#L1238)
    - `if "out-of-band"...` is for handling AIP 2.0 invitations, otherwise, AIP 1.0
    - In either case, call the ACA-Py instance endpoint
    - Since the “--auto-accept-invites” ACA-Py startup parameter is used, the controller just has to wait for the ACA-Py instance to complete the invitation handling, and then get the connection ID ("self.connection_id = ...")

#### User input Processing

- **[Main loop](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/alice.py#L148)** for processing command line input from the user.

#### Send a Basic Message

- The user [chose Option 3](https://github.com/hyperledger/aries-cloudagent-python/blob/467104fab662507e18483abaaf2305bc702bb303/demo/runners/alice.py#L181), send a message.
  - Prompt for the message.
  - “await alice_agent.agent.admin_POST …” invoke ACA-Py to start an instance of the Basic Message protocol ([RFC 0095](https://github.com/hyperledger/aries-rfcs/tree/main/features/0095-basic-message)).

While we won’t go into detail here about the internals of ACA-Py (since developers that write controllers don’t need to do that), for the curious, here are the links to the ACA-Py code for the [connections](https://github.com/hyperledger/aries-cloudagent-python/tree/main/aries_cloudagent/protocols/connections) and [basic message](https://github.com/hyperledger/aries-cloudagent-python/tree/main/aries_cloudagent/protocols/basicmessage) protocol handlers.

## Takeaways

This lab demonstrates:

- Starting agents.
- Generating an (AIP 1.0) invitation using the connection protocol ([RFC 0160](https://github.com/hyperledger/aries-rfcs/tree/main/features/0160-connection-protocol)).
- Processing an invitation.
- Sending a basic message using the basic message protocol ([RFC 0095](https://github.com/hyperledger/aries-rfcs/tree/main/features/0095-basic-message)).
- Receiving a basic message.

One thing you may notice is that the Alice controller is much simpler than the
Faber one, even though it is handling all of the same use cases and options as
Faber. That is because the Faber controller is (for the most part) initiating
the actions, while the Alice controller is responding. With each option that
invokes a capability in a different way (such as using AIP 1.0 vs. AIP 2.0
connection and credential exchange protocols), Faber needs code to pick the one
it will use. Alice on the other hand, handles both options, so it (for the most
part) doesn't need separate code--it just passes what it receives from Faber and
ACA-Py takes care of it. This is a common trait across Aries Frameworks.

This lab provides a good introduction to the construction of a controller for an ACA-Py instance. Developers should focus on connecting what they see on the command line with the code that is making that happen.

While the controllers here are quite simple, we’ve covered a lot of what makes an application a controller. Much of the additional complexity will not be in controller-specific capabilities but rather in the normal complexity of writing a business/web application.

That's it for this lab! Please return to the course.
