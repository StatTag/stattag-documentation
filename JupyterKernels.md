# Jupyter Kernels

## Learning from Examples
### Hydrogen
An Atom-based (using JavaScript) implementation is [Hydrogen](https://github.com/nteract/hydrogen/).  This in turn uses a node module called [kernelspecs](https://github.com/nteract/kernelspecs) to load the list of available kernels.  Note that `kernelspecs` in turn uses [jupyter-paths](https://github.com/nteract/jupyter-paths/blob/master/index.js).

There is also another npm module called [spawnteract](https://github.com/nteract/spawnteract) that launches Jupyter kernels.

#### Enumerating Kernels
`jupyter-paths` will let us know where there's a directory of directories, each representing a kernel.  Within each directory, we'll find a [kernel.json](https://github.com/nteract/kernelspecs/blob/master/lib/traverse.js#L24) file that provides the parameters needed to launch the specific kernel.

For example, on Windows, a [jupyter folder under the user's AppData directory](https://github.com/nteract/jupyter-paths/blob/master/index.js#L158-L160) will be the location.

#### Launching a Kernel
Using `spawnteract`, a new process is created.  Note the [required information](https://github.com/nteract/spawnteract/blob/master/index.js#L53-L64), including the 4 ports needed for ZMQ communication.  This uses [portfinder](https://github.com/indexzero/node-portfinder) to get open ports on the machine.  It looks like the default is to scan the range 8000 - 65535.

Actually running the kernel takes the parameters within the kernel.json file and [launches a new process](https://github.com/nteract/spawnteract/blob/master/index.js#L179) (also shown here):

```
  const runningKernel = execa(argv[0], argv.slice(1), fullSpawnOptions);
```

`argv` comes from the list of parameters in kernel.json.

## .NET options for ZeroMQ
It looks like ICSharpCore [uses NetMQ](https://github.com/SciSharp/ICSharpCore/blob/master/src/ICSharpCore.Kernel/ICSharpCore.Kernel.csproj#L10) for their package, [as does ICSharp](https://github.com/zabirauf/icsharp/blob/master/Kernel/packages.config#L5).

## StatTag Implementation
### JupyterKernelManager
This is a .NET implementation that will manage discovering, launching, communicating with, and shutting down Jupyter kernels on the local system.

#### KernelSpecManager
Responsible for finding all of the Jupyter kernels that are loaded on the local computer.  Will return a list of kernel specifications (KernelSpec) that have the information needed to start each kernel.

#### KernelManager
Each instance will manage a single Jupyter kernel instance.  It requires a KernelSpec (from the KernelSpecManager).  The method `StartKernel` will then invoke the process which the kernel actually runs under.

When the KernelManager starts a kernel, it will also generate a signature key (HMAC SHA-256).  This must be done at the time we are creating and launching kernels, because the shared key must exist in the kernel's connection file.  When a new KernelClient is requested, it will have the key embedded within it.

#### KernelClient
From your program, communication with the kernel is done via a KernelClient.  You can create an instance of the client from the KernelManager's `CreateClient` method.  Once you have the instance of the client, you will need to call `StartChannels` to open up the ZMQ sockets that Jupyter uses for communication (don't forget to to call `StopChannels` when you're all done).

You can run code by calling `Execute`.  This will internally track the blocks of code as you send them, and their respective status within the kernel.  You can check for any outstanding execution requests with `HasPendingExecute` or with `GetPendingExecuteCount`.  You can also block permanently or temporarily (specifying a timeout) to wait for these via `WaitForPendingExecute`.

###### Sockets
The following sockets are created and used by the client to communicate with the kernel.  Note that ZMQ uses paired types of sockets for communications.  If you look at information about Jupyter kernels, you will see different socket types listed.  The order in the following table is the order in which the sockets are created.

|Channel|Socket Type|Description*|
|-------|-----------|-----------|
|Shell  |Dealer     |Receives our code to execute|
|IoPub  |Subscriber |All kernel side-effects received here (e.g., stdout, stderr)|
|StdIn  |Dealer     |Any interactive (stdin) input from our client (Not planned for use in StatTag)|
|Heartbeat|Request  |Makes sure we're still connected to the kernel|
|Control|Dealer     |Like Shell, but for important messages (e.g., shutdown)|

\* Full details available on the [Messaging in Jupyter](https://jupyter-client.readthedocs.io/en/stable/messaging.html) page

## Other Notes
**Launching R kernel**
```
"c:/PROGRA~1/R/R-35~1.2/bin/x64/R" --slave -e IRkernel::main() --args C:\Development\jupyter-test\kernel.json
```
