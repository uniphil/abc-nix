# ABC Nix

How to do stuff with NixOS, so I don't forget.


## Add and remove packages from nixpkgs

### System-wide

1. As root, edit `/etc/nixos/configuration.nix`:

    ```nix
      environment.systemPackages = with pkgs; [
        abiword
        acpi
        alsaUtils
        ...
    ```

2. Rebuild the system

    ```bash
    $ sudo nixos-rebuild switch
    ```


## Update stuff

1. Update the package stuff

    ```bash
    $ sudo nix-channel --update
    ```

2. Rebuild with updated stuff

    ```bash
    $ sudo nixos-rebuild switch
    ```
