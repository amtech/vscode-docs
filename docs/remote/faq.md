---
Order: 6
Area: remote
TOCTitle: FAQ
PageTitle: Visual Studio Code Remote Development Frequently Asked Questions
ContentId: 66bc3337-5fe1-4dac-bde1-a9302ff4c0cb
MetaDescription: Visual Studio Code Remote Development Frequently Asked Questions (FAQ) for SSH, Containers, and WSL
DateApproved: 5/2/2019
---
# Remote Development FAQ

❗ **Note:** The **Remote Development extensions** require **[Visual Studio Code Insiders](http://code.visualstudio.com/insiders)**.

---

This article covers frequently asked questions for each of the **Visual Studio Code Remote Development** extensions. See the [SSH](/docs/remote/ssh.md), [Containers](/docs/remote/containers.md), and [WSL](/docs/remote/wsl.md) articles for more details on setting up and working with each of their respective capabilities.

## General

### What is Visual Studio Code Remote Development?

The Visual Studio Code [Remote Development](https://aka.ms/vscode-remote/download/extension) extension pack allows you to open any folder in a container, on a remote machine (via SSH), or in the Windows Subsystem for Linux and take advantage of VS Code's full feature set. This means that VS Code can provide a local-quality development experience — including full IntelliSense (completions), debugging, and more — regardless of where your code is located or hosted.

### What advantages does VS Code Remote Development provide over local editing?

Some benefits of remote development include:

* Being able to edit, build, or debug on a different OS than you are running locally.
* Being able to develop in an environment that matches the target deployment environment.
* Using larger or more specialized hardware than your local machine for development.
* The ability to edit code stored in another location, such as in the cloud or at a customer site.
* Sandboxing of developer environments to avoid conflicts, improve security, and speed up on-boarding.

Compared to using a network share or synchronizing files, VS Code Remote Development provides dramatically better performance along with better control over your development environment and tools.

### How do the Remote Development extensions work?

Visual Studio Code Remote Development allows your local VS Code installation to transparently interact with source code and runtime environments on other machines (whether virtual or physical) by moving the execution of certain commands to a "remote server". The **VS Code Server** is quickly installed by VS Code when you connect to a remote endpoint and can host extensions that interact directly with the remote workspace, machine, and file system.

![Architecture summary](images/troubleshooting/architecture.png)

See [Supporting Remote Development](/api/advanced-topics/remote-extensions.md) for additional details about extensions.

### How do the Remote Development extensions secure access to a remote machine, VM, or container?

Visual Studio Code Remote Development uses existing, well known transports like [secure shell](https://en.wikipedia.org/wiki/Secure_Shell) to authenticate and secure traffic. No ports need to be publicly opened beyond those used by these well-known, secure transports.

The VS Code Server that is injected runs as the same user you used to sign into the machine, ensuring that VS Code and its extensions are not given improper elevated access without permission. The server is started and stopped by VS Code and is not wired into any user or global login or startup scripts. VS Code manages the server's lifecycle so you do not need to worry about whether or not it is running.

### Can the VS Code Server be installed or used on its own?

No. The VS Code Server is a component of the Remote Development extensions and is managed by a VS Code client. It is installed and updated automatically by VS Code when it connects to an endpoint and is not intended or [licensed](https://go.microsoft.com/fwlink/?linkid=2077057) for use by other clients.

### What are the connectivity requirements for VS Code Server?

The VS Code Server requires outbound HTTPS (port 443) connectivity to `update.code.visualstudio.com` and `marketplace.visualstudio.com`. All other communication between the server and the VS Code client is accomplished through the following transport channels depending on the extension:

* SSH: An authenticated, secure SSH tunnel.
* Containers: An authenticated, random port automatically exposed via the Docker CLI.
* WSL: An authenticated, random local TCP port.

### What Linux packages or libraries need to be installed on a host to use Remote Development?

Most Linux distributions will not require additional dependency installation steps. For SSH, Linux hosts need to have Bash (`/bin/bash`), `tar`, and either `curl` or `wget` installed and those utilities could be missing from certain stripped down distributions. [Alpine Linux](https://alpinelinux.org) is currently not supported.

### Can the Docker extension run as a remote "workspace" extension?

The Docker extension is configured to run as a local "UI" extension by default. This enables the extension to work with your local Docker installation when you are developing inside a container. However, you may want to use the extension with a Docker Machine installed on a remote host instead. Fortunately, you can configure the Docker extension to run on the host by adding the following to `settings.json`:

```json
"remote.extensionKind": {
    "peterjausovec.vscode-docker": "workspace"
}
```

### Can I install individual extensions instead of the extension pack?

Yes. The [Remote Development](https://aka.ms/vscode-remote/download/extension) extension pack provides a convenient way for you to access all of the latest remote capabilities as they are released. However, you can always install the individual extensions from the Marketplace or VS Code Extensions view.

* [Remote - SSH](https://aka.ms/vscode-remote/download/ssh)
* [Remote - Containers](https://aka.ms/vscode-remote/download/containers)
* [Remote - WSL](https://aka.ms/vscode-remote/download/wsl)

## Remote - Containers

### Do "dev container definitions" define how an application is deployed?

No. A development container defines an environment in which you develop your application before you are ready to deploy. While deployment and development containers may resemble one another, you may not want to include tools in a deployment image that you use during development.

The [vscode-dev-containers repo](https://aka.ms/vscode-dev-containers) includes a set of dev container definitions for some common development environments. You can also [attach to a running container](/docs/remote/containers.md#attaching-to-running-containers) without setting up a dev container definition, if you prefer to use an alternate container build or deployment workflow.

### Do "dev containers definitions" define how an application is built? Like Buildpacks?

No. The [Buildpacks](https://buildpacks.io/) concept focuses on taking source code and generating deployable container images through a series of defined steps. A dev container is an environment in which you can develop your application before you are ready to build. They are therefore complementary concepts.

### Why do some commands invoked from the Docker extension fail?

Using the Docker extension from a VS Code window opened in a container has some limitations. Most containers do not have the Docker command line installed. Therefore commands invoked from the Docker extension that rely on the Docker command line, for example **Docker: Show Logs**, fail. If you need to execute these commands, open a new local window and use the Docker extension from this VS Code window or [set up Docker inside your container](https://aka.ms/vscode-remote/samples/docker-in-docker).

## Extensions authors

### As an extension author, what do I need to do?

The VS Code extension API abstracts away local/remote details so most extensions will work without modification. However, given extensions can use any node module or runtime they want, there are situations where adjustments may need to be made. We recommend you should test your extension (particularly in a container) to be sure that no updates are required. See [Supporting Remote Development](/api/advanced-topics/remote-extensions.md) for details.

### Can an extension access local resources or APIs when a user is connected remotely?

When VS Code connects to a remote environment, extensions are classified as either **UI** or **Workspace** extensions. UI Extensions run in a **local extension host**, can contribute UI or personalization features (for example themes), and have access to local files or APIs. Workspace extensions run in a **remote extension host** with the workspace and have full access to the source code, remote filesystem, and remote APIs. While Workspace extensions do not focus on UI customization, they can contribute explorers, views, and other UI elements as well.

When a user installs an extension, VS Code attempts to infer the correct location and install it based on its type. Extensions that do not need to run remotely like themes and other UI customizations are automatically installed on the UI side. All others are treated as Workspace extensions since they are the most full-featured. However, extension authors can also override this location with an `extensionKind` property in `package.json`.

See [Supporting Remote Development](/api/advanced-topics/remote-extensions.md)for additional details.

## License and privacy

### Location

You can find the licenses for the VS Code Remote Development extensions here:

* [Remote-SSH License](https://marketplace.visualstudio.com/items/ms-vscode-remote.remote-ssh/license)
* [Remote-WSL License](https://marketplace.visualstudio.com/items/ms-vscode-remote.remote-wsl/license)
* [Remote-Containers License](https://marketplace.visualstudio.com/items/ms-vscode-remote.remote-containers/license)

### Why aren't the Remote Development extensions using the MIT License?

The VS Code Remote Development extensions are available under Microsoft pre-release licenses, similar to other service-based extensions such as [Visual Studio IntelliCode](https://marketplace.visualstudio.com/items/VisualStudioExptTeam.vscodeintellicode/license) and [Visual Studio Live Share](https://marketplace.visualstudio.com/items/MS-vsliveshare.vsliveshare-pack/license). A Microsoft license (instead of MIT, for example) makes it easier for us to license certain features of the product, such as access to the Visual Studio Marketplace (section 1.d), the re-licensing of third-party components (section 1.c), and telemetry data (section 4).

### Why aren't the Remote Development extensions or their components open source?

The Visual Studio Code Remote Development extensions and their related components will use an [open planning, issue, and feature request process](https://aka.ms/vscode-remote/feedback), but are not currently open-source. As with other service-based extensions such as [Visual Studio LiveShare](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare-pack) and [IntelliCode](https://marketplace.visualstudio.com/items/VisualStudioExptTeam.vscodeintellicode), we have made the decision to keep them closed source.

### Will you charge for the extensions once they exit "Preview"?

No, they will remain free of charge. In the future, we may provide "premium" developer services, which provide additional functionality, but the extensions will be free.

### Can I host the VS Code Server in my Service?

No. The extensions will automatically install the proper version of the server in the host (based on commit) so you cannot pre-install the server as it would quickly become out of sync with the extensions. Furthermore, the license states that you may not "provide the software as a stand-alone or integrated offering or combine it with any of your applications for others to use" which means you are not permitted to use the Remote Development extensions with a hosted development offering. You are free to use the Remote Development extensions with personal or work computers, internally hosted Virtual Machines, or Container instances as long as you do not expose the services as a publicly available development offering.

### GDPR and VS Code Remote Development

The VS Code Remote Development extensions follow the GDPR policies as Visual Studio Code itself. See the [general FAQ](/docs/supporting/faq.md#gdpr-and-vs-code) for more details.

## Questions or feedback

Have a question or feedback?

* See [Tips and Tricks](/docs/remote/troubleshooting.md).
* Search on [Stack Overflow](https://stackoverflow.com/questions/tagged/vscode-remote).
* Add a [feature request](https://aka.ms/vscode-remote/feature-requests) or [report a problem](https://aka.ms/vscode-remote/issues/new).