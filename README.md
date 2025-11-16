# Rootstock

Rootstock is the hub-deployment layer of the [Arborist Suite](https://github.com/jwfraustro/arborist). Together with [Scion](https://github.com/jwfraustro/scion) and [Chainsaw](https://github.com/jwfraustro/chainsaw), Rootstock is capable of turning a single compromised host into a centralized reverse-shell management hub.

Rootstock automates:

- Obtaining a shell on the target host
- Setting up necessary system libraries
- Deploying and configuring rshell services to phone home
- Orchestrating mass reverse-shell grafting of target hosts via Scion
- Creating an rshell back home to your machine
- Returning all connected shells to calling scripts for management

Where Scion creates the *grafts* of the tree, Rootstock creates the *branches*.

## Architecture Diagram
```
Scion    Scion    Scion   Scion <--- RShells
    \    /          \     /
   Rootstock        Rootstock   <--- Rootstock Hubs
       \__\          _/_/
          \\        //
           \\      //
           Arborist             <--- RShell Manager (Optional)
              |:|
           ../|:|\.-.
```

## What's Rootstock?
In horticulture, a [rootstock](https://en.wikipedia.org/wiki/Rootstock) is the part of a plant onto which new cuttings are grafted.

In the Arborist Suite, Rootstock is responsible for creating the hub that later reverse-shells will be grafted onto. After the initial deployment of the Rootstock hub, Rootstock can then leverage Scion to mass-deploy reverse-shell grafts to other target hosts.

Only your connection to the Rootstock hub is visible to network monitoring; all grafted reverse-shells connect back to the Rootstock hub, which in turn connects back to you. If the Roostock hub becomes compromised you can perform lift-and-shift operations to move all grafts to a new Rootstock hub using the Arborist tool.

## Installation
It is highly recommended that you clone this repository and use the [Greybel](https://github.com/ayecue/greybel-js) CLI tool or the [Greybel VSCode extension](https://github.com/ayecue/greybel-vs) to build the Rootstock source and import it into Grey Hack.

The [gh-corelib](https://github.com/jwfraustro/gh-corelib), [scion](https://github.com/jwfraustro/scion), and [chainsaw](https://github.com/jwfraustro/chainsaw) libraries are provided as git submodules. Be sure to clone the repository with the `--recurse-submodules` flag to ensure that these dependencies are included.

```
git clone --recurse-submodules http://github.com/jwfraustro/rootstock
```

I really don't suggest downloading or copying the source code manually, as it may lead to headaches when it comes to sorting out dependencies.

## Usage
Rootstock is designed to be controlled by another script and **does not run from the command line**. A very basic example:
```
import_code("rootstock.src")

rootstock = new Rootstock
rootstock.hub_ip = "123.44.12.7"
rootstock.ip_group = ["10.0.0.5", "10.0.0.6"]

rootstock.DeployHub()
rootstock.DeployGrafts()

print("Connected grafts:")
rootstock.CheckGrafts()
```

To deploy a Rootstock hub, you must import the 'rootstock.src' module into your Grey Hack script. At minimum you will need to instantiate a `Rootstock` object and either run the `init()` method to interactively set up the Rootstock object, or manually set the target hub's IP address before calling the `DeployHub()` method.

```miniscript
import_code("rootstock.src")

rootstock = new Rootstock
rootstock.init() // to set up interactively
// or
rootstock.hub_ip = "1.2.3.4" // the target address for the Rootstock hub
// rootstock.hub_password = "password" // if the password to the target host's SSH is known
rootstock.DeployHub()
```

If you wish to deploy reverse shell grafts via the Rootstock hub, you may provide a list of ip addresses and call the `DeployGrafts()` method.

```miniscript
import_code("rootstock.src")

rootstock = new Rootstock
rootstock.init()
rootstock.ip_group = ["1.1.1.1", "1.1.1.2", ...]

rootstock.DeployHub()
rootstock.DeployGrafts()

rootstock.CheckGrafts() // print status of grafts
```

## Details
### Deployment

Rootstock uses SSH to connect to the target host and deploy the Rootstock hub. If the password to the target host is not known, Rootstock will automatically attempt to exploit any vulnerabilities found on the target host, and acquire root credentials using [chainsaw](https://github.com/jwfraustro/chainsaw).

Once connected, Rootstock will attempt to copy `aptclient.so` and `metaxploit.so` system libraries to the target host. If it succeeds, it will deploy an RCE script to set up an rshell server on the target host as well as a script to fetch all connected graft shells.

### Grafting
After the Rootstock hub is deployed, Rootstock can then attempt to deploy reverse-shell grafts to other target hosts using Scion to automatically self-elevate and set up rshell clients. Alternatively, if you prefer or need to manually set up grafts, you can simply point any rshell client to connect back to the Rootstock hub.

### Checking Grafts
Rootstock provides a `CheckGrafts()` method that will simply print the status of all grafts connected to the Rootstock hub through an RCE script.

### Fetching Shells
Because rshell connections (at the time of this script's creation) are not automatically able to be passed back to the calling script through multiple connections, Rootstock creates a `get_grafts` executable on the hub host sets the graft shells into the `get_custom_object.shells` property. The calling script can then access the connected graft shells through this property. Additionally, it means that the calling script never has to directly connect to the grafts themselves, as all commands can be routed through the Rootstock hub.

For an example of how this works in practice, check the `branch.src` file in the Arborist Suite repository.

### On Complexity
Rootstock is a fairly complex concept to grasp, and its power is best demonstrated when used in conjunction with the Arborist tool.

## Support Notice
Rootstock is provided as-is. I no longer play Grey Hack, and future maintenance is unlikely. I cannot guarantee that Rootstock will work with future versions of Grey Hack. This project is simply provided as a reference for those interested.

Fun fact: Scion, Rootstock, and Arborist were used to create a network of over 7000 rshell connections in the creation of my (now defunct) GreyHack Network Map project.

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.