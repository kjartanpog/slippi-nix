# slippi-nix

This project is a Flake which exposes NixOS modules for ensuring your GameCube
controller adapter is performing as well is it can be and Home Manager modules
for installing and configuring the Slippi Launcher using NixOS-compatible
versions of both the playback and netplay builds of Dolphin (Ishiiruka).

**NOTE**: At this time, this Flake only supports the `x86_64-linux` `system`.

# Usage

The simplest usage is to simply run the netplay package in this Flake:

```shell
nix run github:lytedev/slippi-nix#slippi.netplay
```

However, for the optimal experience, you will want to ensure the host has the
necessary configuration to handle a GameCube controller adapter. Additionally,
you likely want to make use of the Slippi Launcher for managing and viewing
your replays.

In your flake, you may optionally import the NixOS module for the overclocked
adapter and you must import the Home Manager module for the Slippi Launcher. You
will also need to specify where your Melee ISO is. Here is an example:

```nix
{
  inputs.slippi.url = "github:lytedev/slippi-nix";
  inputs.nixpkgs.url = "...";

  outputs = {slippi, nixpkgs, ...}: {
    nixosConfigurations = nixpkgs.lib.nixosSystem {
      system = "...";
      modules = [
        slippi.nixosModules.default
        {
          home-manager = {
            # ... snip -- see Home Manager's documentation for details
            users.YOUR_USERNAME = {
              imports = with outputs.homeManagerModules; [
                slippi.homeManagerModules.default
                {
                  # use your path
                  slippi.launcher.isoPath = "/home/user/Downloads/melee.iso";
                }
              ];
            };
          };
        }
      ];
    };
  };
}
```

There are other configuration options if you take a look at the flake's source.
Most useful to me is:

```nix
slippi.launcher.launchMeleeOnPlay = false;
```

As I prefer to be able to tweak Dolphin's settings before diving right in.

# Updates

Note that this does _not_ support the auto-updates performed by Slippi Launcher.
This Flake exposes extra configuration fields for specifying the version of the
AppImage you want along with the hash like so:

```nix
{
  home-manager.users.YOUR_USERNAME = {
    slippi.launcher.netplayVersion = "3.4.0";
    slippi.launcher.netplayHash = "sha256-d1iawMsMwFElUqFmwWAD9rNsDdQr2LKscU8xuJPvxYg=";
    slippi.launcher.playbackVersion = "3.4.0";
    slippi.launcher.playbackHash = "sha256-d1iawMsMwFElUqFmwWAD9rNsDdQr2LKscU8xuJPvxYg=";
  };
}
```

So when a Slippi update is released, you can usually bump the version to match
and update the hash with whatever `nix` says it is.

# To Do

It would be nice if Home Manager weren't strictly necessary. At the moment,
I believe the config file _could_ be created via a NixOS module (using
`systemd.tmpfiles.rules` or something?), but the ideal scenario would be to
instead either be to specify to the launcher where the netplay and playback
binaries are without the configuration file -- perhaps via command line
arguments or environment variables.

If this were true, you could simply `nix run github:lytedev/slippi-nix#netplay`
and start playing right away, which is ideal!
