# Systemd and Service Management 

## Learning goals

- Learn what `systemd` is and why it exists
- Gain a basic understanding of how `systemd` services are created and where they are stored
- Learn how to use the `systemctl` command

## Introduction

`systemd` is a software suite that is bundled in a wide array of *Linux* distributions, including the most popular ones like *Ubuntu*, *Fedora*, *Debian*, etc. It is what is known as a *system* and *service* manager, and is responsible for a lot of crucial aspects of the operating system, ranging from service management to initializing the operating system.

`systemd` originated as a replacement for another init system, `System V`, allowing for faster boot times and features like dependency management and process tracking. An *init system* is simply the first process that starts up in a *Linux* system and is responsible for initializing a lot of the processes and components you would expect to be initialized when booting your system (e.g. hardware devices, file system, network devices).

In this lesson, we will learn how `systemd` allows us, the end-user, to create and manage services on our system.

## Services

**Services**, also called **units**, are how `systemd` organizes itself and knows what to run. 

There are several types of services, ranging from a standard general-purpose service to a `socket` or `device` service which have more specific applications. In this lesson, we will be going over the standard types of service.

Each service is identified by a unique name, and has a respective configuration file that follows a specific syntax containing all the information regarding the service.

The service files typically have the `.service` extension and can be located in either of the following directories:

- `/etc/systemd/system`: Services in this directory are ran as the `root` user
- `~/.config/systemd/user`: Services in this directory are ran as the currently logged-in user, so they might not have `root` permissions

> **Note:** Try using `ls` and then `cat`/`nano` to view the contents of some of the existing scripts to see how they work and interact with the system!

Let's start exploring service configuration files by creating a simple service that runs at startup and stores in a file (let's call it `/usr/logs/start-time.txt`) the time when it was ran:

`/etc/systemd/system/start-time.service`:

```bash
[Unit]
Description=Store in /usr/logs/start-time.txt when the system last started up

[Service]
Type=oneshot
ExecStart=/bin/bash -c "date > /usr/logs/start-time.txt"
```

The first section, `[Unit]`, contains metadata related to the service. In this case, we're only storing a description of what the service does, but there are a lot of fields that could go here, including:

- `After=`: The service will start *after* the services listed here
- `Before=`: The service will start *before* the services listed here
- `Conflicts=`: The service cannot be run at the same time as other services listed here

> **Note:** You can repeat these directives; for instance, you could have three lines with `After=` to list three services that go before this one.

The second section stores the meat of the service. The `Type=` directive specifies the *execution* type, of which there are many, including:

- `simple`: This is the default service type and is suitable for simple services that don't fork into the background
- `forking`: This service type is designed for services that fork into the background after starting 
- `oneshot`: This service type is designed for services that are expected to run once and then exit

In our case, since all we are interested in is writing to a file once and then exiting, we want to use the `oneshot` type.

Finally, `ExecStart=` gets set to what the service *actually* does, and it normally links to an executable binary. Since we want to run a shell command to write to the file, we execute our bash shell with `/bin/bash`, which is where it is stored. We use the `-c` parameter to tell it that the text that follows should be executed as a command, and lastly the output of the `date` command is stored (`>`) into our file, `/usr/logs/start-time.txt`.

> **Note:** Remember, if you don't know what an argument or command does, you can always run `man` (manual) to get a description! Try `man bash` to see what it says about the `-c` parameter.

## `systemctl`

The command you will be using for the most part when interacting with `systemd` is the `systemctl` command. This command allows you to `start`, `stop`, `enable`, `disable`, and view statuses and logs for services:

```bash
sudo systemctl start start-time.service
sudo systemctl stop start-time.service
sudo systemctl enable start-time.service
sudo systemctl disable start-time.service
sudo systemctl status start-time.service
```

When you `enable` a service, it will run every time the system boots, but not when you ran the command. Using `start` will launch the service, but it will not make it run every boot.

If you try using these commands before restarting the system, you will be prompted with errors. That's because after adding or modifying any of the `systemd` configuration files, we need to manually refresh it:

```bash
sudo systemctl daemon-reload
```

Now the commands should work!

## Conclusion

That's about it! `systemd` is an integral part of many *Linux* distributions, and responsible for managing a lot of crucial components in servers. It's important to learn how it works early on so you can tell when 

To learn more about `systemd` in detail, or if you just want a reference sheet, you can find a great resource [here](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files).