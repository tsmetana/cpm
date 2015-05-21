# Container Package Manager

The `cpm` script is a proof-of-concept of managing application or OS containers using the system package management tools.
It is focused on systemd-nspawn type of containers and uses dnf/rpm for package management. The script is still
work-in-progress and might not work or even cause damage to your system.

The TODO lists in every use-case outline the PoC completeness.

## Intended Use-Cases

The script serves as a wrapper to `dnf` and makes the systemd-nspawn containers somewhat easier to manage. However the
first four use cases are nothing more than semantic shortcuts: it's possible with plain `dnf` (or `yum`) just with
slightly more typing.

### 1. Container Installation

Install a package into a container specified by its name:

```
cpm install -m "my_machine" httpd
```

This will:
* Find the location where the container root filesystems reside (`/var/lib/machines` by default); optionally the location can be
specified explicitly (`-D` option).
* Create the new root directory if not exists already (`/var/lib/machines/my_machine` in the example).
* Install the `httpd` package and all its dependencies using the `/var/lib/machines/my_machine` as the installation root

TODO:
* Add user interactivity (confirmations; progress)
* Do not install %doc files
* Automatically clean up the dnf cache inside the container
* Allow the source repository selection

### 2. Updating Container Content

To update the `httpd` package inside the `my_machine` container:

```
cpm update -m "my_machine" httpd
```

To update all the packages inside the `my_machine` container:

```
cpm update -m "my_machine" httpd
```

Similarly to the installation the script tries to find the container itself at "well known" locations or the location
optionally specified by the `-D` command line switch.

TODO:

* Automatically clean up the dnf cache inside the container

### 3. Remove Container Content


```
cpm remove -m "my_machine" httpd
```

This command will (unsurprisingly) remove the httpd package from the "my_machine" container.


### 4. Inspect Container Content

To get a list of "my_machine" packages:

```
cpm list -m "my_machine"
```

### 5. Create a Container "Definition" Package

The main point of the script is to easily build an package that would contain all the modifications made to the container
root directory.

```
cpm checkinstall -m "my_machine"
```

This command will scan the "my_machine" subdirectory for any altered or added files and create an RPM package of them.
Additionally it would identify all the leaf packages of the container and add them as requirements for the new package.
Installation of the newly created package (see the use-case #1) would then re-create the container. There is still a lot
missing for this to work completely and correctly.

TODO:
* Figure out how to treat the modified and removed files
