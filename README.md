# Lisk Hackathon 2023

This readme will cover some details for [Lisk Hackathon 2023](https://hackathon.lisk.com/).

## Lisk SDK Version

We are going to use Lisk SDK version [6.1.0-beta.1](https://github.com/LiskHQ/lisk-sdk/releases/tag/v6.1.0-beta.1)
```
npm i lisk-sdk@6.1.0-beta.1
```

## Lisk Commander Version

We are going to use the Lisk Commander version [6.1.0-beta.1](https://www.npmjs.com/package/lisk-commander/v/6.1.0-beta.1)
```
npm i -g lisk-commander@6.1.0-beta.1
```

## Resources

Along with a series of presentations during the hackathon sharing knowledge including Live demos and code walk-throughs.

Lisk SDK documentation: https://lisk.com/documentation/lisk-sdk/v6/index.html

## Getting Started with Lisk Blockchain Client

This project was bootstrapped with [Lisk SDK](https://github.com/LiskHQ/lisk-sdk)

1. `git clone https://github.com/LiskHQ/lisk-hackathon.git`
2. `cd lisk-hackathon`
3. `npm install && npm run build`

### Start a node

```
./bin/run start
```

Note: To connect to the network created for hackathon you can use `https://hackathonnet-seed-01.liskdev.net/` node. You can update config file, network `{ seedPeers: [{"ip": "hackathonnet-seed-01.liskdev.net", "port": 7667 }] }`.

#### Enable plguins and RPC API using flags

You can use either `--api-ipc` to enable IPC API, or `--api-http` to enable HTTP API or `--api-ws` to enable WS API or you can use all to enable access to all the modes of communication.

```
./bin/run start --api-ipc --api-http --enable-dashboard-plugin
```

### Add a new module

```
lisk generate:module ModuleName
// Example
lisk generate:module token
```

### Add a new command

```
lisk generate:command ModuleName Command
// Example
lisk generate:command token transfer
```

### Add a new plugin

```
lisk generate:plugin PluginName
// Example
lisk generate:plugin httpAPI
```

## Learn More

You can learn more in the [documentation](https://lisk.com/documentation/lisk-sdk/).

## Bonus content

This section is for those who have already created a sidechain application and want to have cross-chain functionality. If you want to dive deep into what Lisk Interoperability is you can visit this blog https://lisk.com/blog/posts/what-blockchain-interoperability.

First, make your existing module interoperable or create a new interoperable module from scratch.

### Create Interoperable module

This guide gives step-by-step instructions for adding an interoperable module.

Before jumping on to the steps, first understand what is an interoperable module. Interoperable module is a module that not only has state transition functions via Methods, Commands, and Block lifecycle hooks but also can do state transitions on every cross-chain message (aka CCM). An interoperable module will have 2 main components.

- Cross Chain Command(ccCommand): Similar to commands it will have a schema, verify and execute methods and each CCM triggers the logic associated with <module, ccCommand>

- Cross Chain Methods(ccMethod): Similar to block lifecycle hooks, these hooks are called during CCM execution (when the apply/forward internal method is called by a CCU transaction that contains multiple CCMs). They can be implemented by any interoperable module and these hooks will be executed for each module when doing CCM processing.

  - `beforeRecoverCCM`: Before recovery (called on each module)

  - `recover`: Performs recovery (called by a specific module ccm is pointing to)

  - `verifyCrossChainMessage`: Before calling beforeCrossChainCommandExecute on a CCM (called on each module)

  - `beforeCrossChainCommandExecute`: Before executing a CCM (called on each module)

  - `afterCrossChainCommandExecute`: After executing a CCM (called on each module)

  - `beforeCrossChainMessageForwarding`: Before forwarding a CCM (called on each module)

#### How to create and register an interoperable module?

1. Creating an interoperable module is similar to creating a custom module. We can also use Lisk Commander to first generate a module and its related file by calling.,

```ts
lisk generate:module <name>
```

2. Instead of extending your module’s class with BaseModule, extend it using BaseInteroperableModule. This will help you to identify missing information in TypeScript.

3. Once you have all the files and folder for your module, you need to add the components as discussed in the previous section.

- Add a folder `cc_commands` under your `module_name` folder.

- Add cc_method.ts file under your module_name folder.

4. Under `cc_commands` add any `ccCommand` you want for your interoperable module. One example is the Token module’s `crossChainTransfer` ccCommand. Now add one or multiple `ccCommands` to an array with the property name `crossChainCommand` of your module class which is also inherited from the BaseInteroperableModule class.

5. In the `cc_methods.ts` file created at step `3.b`, create a class that extends `BaseCCMethod`. If you want to add logic to any CCM processing lifecycle hooks for Cross Chain Methods, then add that logic under a specific hook. After you have added your logic, import and assign this class’s object in the module.ts file to the crossChainMethod property.

6. Most probably you will be using the interoperability method from your interoperable module to send or error a CCM or get some info. In order to use it, implement the `addDependencies` method on your module class and assign `interoperableMethod` to a property declared within your module class. Below is an example,

```ts
private _interoperabilityMethod!: InteroperabilityMethod;
public addDependencies(interoperabilityMethod: InteroperabilityMethod, feeMethod: FeeMethod) {
		this._interoperabilityMethod = interoperabilityMethod;
}
```

7. Now your module is ready and you can register it to your application by modifying your modules.ts and calling the below method to register your interoperable module to the interoperability module.

```ts
app.registerInteroperableModule(<INTEROPERABLE_MODULE_NAME>)
```

That’s it, your interoperable will be working with your sidechain application.

### Register your sidechain

[Sidechain registration](https://lisk.com/documentation/beta/understand-blockchain/interoperability/sidechain-registration-and-recovery.html#sidechain-registration) guide will help you to register your sidechain application on mainchain and mainchain on your sidechain.

### Setup relayer node

[Setting up a relayer node](https://lisk.com/documentation/beta/run-blockchain/setup-relayer.html) guide will help you with setting up a relayer node that will send cross-chain info via cross-chain update transactions.
In order to have bidirectional updates you need to setup one relayer node for sidechain->mainchain and another from mainchain->sidechain. To achieve this you simply have to run one of your sidechain nodes and one mainchain node with chain connecter plugin enabled.

Now you are all set with your interoperability feature and you can make cross-chain transactions to see and test your cross-chain functionality. You can follow [Sidechain Registration & Recovery](https://lisk.com/documentation/beta/understand-blockchain/interoperability/sidechain-registration-and-recovery.html#life-cycle-of-a-sidechain) to follow all the steps mentioned here in detail.
