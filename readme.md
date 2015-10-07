# ABC Nix

How to do stuff with NixOS, for mere mortals like me, so I don't forget.


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

## Override a package

_The Atom editor, in this case_

The [original package definition](https://github.com/NixOS/nixpkgs/blob/release-15.09/pkgs/applications/editors/atom/default.nix) looks like this:

```nix
{ stdenv, fetchurl, buildEnv, makeDesktopItem, makeWrapper, zlib, glib, alsaLib
, dbus, gtk, atk, pango, freetype, fontconfig, libgnome_keyring3, gdk_pixbuf
, cairo, cups, expat, libgpgerror, nspr, gconf, nss, xorg, libcap, systemd
}:

let
  atomEnv = buildEnv {
    name = "env-atom";
    paths = [
      stdenv.cc.cc zlib glib dbus gtk atk pango freetype libgnome_keyring3
      fontconfig gdk_pixbuf cairo cups expat libgpgerror alsaLib nspr gconf nss
      xorg.libXrender xorg.libX11 xorg.libXext xorg.libXdamage xorg.libXtst
      xorg.libXcomposite xorg.libXi xorg.libXfixes xorg.libXrandr
      xorg.libXcursor libcap systemd
    ];
  };
in stdenv.mkDerivation rec {
  name = "atom-${version}";
  version = "1.0.4";

  src = fetchurl {
    url = "https://github.com/atom/atom/releases/download/v${version}/atom-amd64.deb";
    sha256 = "0jki2ca12mazvszy05xc7zy8nfpavl0rnzcyksvvic32l3w2yxj7";
    name = "${name}.deb";
  };

  buildInputs = [ atomEnv makeWrapper ];

  phases = [ "installPhase" "fixupPhase" ];

  installPhase = ''
    mkdir -p $out
    ar p $src data.tar.gz | tar -C $out -xz ./usr
    substituteInPlace $out/usr/share/applications/atom.desktop \
      --replace /usr/share/atom $out/bin
    mv $out/usr/* $out/
    rm -r $out/share/lintian
    rm -r $out/usr/
    patchelf --set-interpreter "$(cat $NIX_CC/nix-support/dynamic-linker)" \
      $out/share/atom/atom
    patchelf --set-interpreter "$(cat $NIX_CC/nix-support/dynamic-linker)" \
      $out/share/atom/resources/app/apm/bin/node
    wrapProgram $out/bin/atom \
      --prefix "LD_LIBRARY_PATH" : "${atomEnv}/lib:${atomEnv}/lib64"
    wrapProgram $out/bin/apm \
      --prefix "LD_LIBRARY_PATH" : "${atomEnv}/lib:${atomEnv}/lib64"
  '';

  meta = with stdenv.lib; {
    description = "A hackable text editor for the 21st Century";
    homepage = https://atom.io/;
    license = licenses.mit;
    maintainers = [ maintainers.offline ];
    platforms = [ "x86_64-linux" ];
  };
}

```

We'll upgrade the version of Atom to the latest, 1.0.19. As you can see, it's doing lots of fancy packaging stuff. I don't understand it and don't want to rewrite it, so we'll override just the parts we care about instead.

First, grab [the tarball from github](https://github.com/atom/atom/releases), and compute the sha256sum:

```bash
$ curl -L https://github.com/atom/atom/releases/download/v1.0.19/atom-amd64.deb | sha256sum
```

I get `77534b5c...` -- note it, we'll need it later!

We'll edit ~/.nixpkgs/config.nix to override atom for our user account. We'll increment the version number, give it the new release url, and the sha256sum we just computed.

`~/.nixpkgs/config.nix`
```nix
with import <nixpkgs> {};

{
  packageOverrides = pkgs: rec {
    atom = let
      version = "1.0.19";
      name = "atom-${version}";
    in pkgs.stdenv.lib.overrideDerivation pkgs.atom (oldAttrs: {
      version = version;
      name = name;
      src = fetchurl {
        url = "https://github.com/atom/atom/releases/download/v${version}/atom-amd64.deb";
        sha256 = "77534b5c706cb2a80eb50ccd1271fcd9d4489ead954d4c08d1960371b8dcfcce";
        name = "${name}.deb";
      };
    });
  };
}
```

Line-by-line, what I've been able to figure out:

1 `with import <nixpkgs> {};` puts some stuf in scope we need. I was getting errors about `fetchurl` not being defined until I added this.

4 `  packageOverrides = pkgs: rec {` straight out of [Modifying Packages](https://nixos.org/wiki/Nix_Modifying_Packages#Creating_a_config.nix_File) on the wiki

5 `    atom = let` I had to bind these variables in the let part of a `let` / `in` expression.

9 `      version = version;` (and `name`) Puts the variables we bound into the record I guess?

11 `      src = fetchurl {` Put in our new url and sha256sum for `fetchurl`

14 `        name = "${name}.deb";` Would be nice to leave this from the original, but I don't know how, so, copy-pasta.

**With all that done, I can `nix-env -i atom`, and nix gets me Atom v1.0.19!**
