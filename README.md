How to use this repository —

- First setup the version you'd like to build

    ```
    $ export GO_VERSION=16.9.0-4001
    ```

- Then run the following to download and build the installers. This may take a few minutes to download depending on your internet connection.

    ```
    $ vagrant up package --provider virtualbox
    ```

- Once the installers, are built, run the following to bring up another virtualbox instance to verify that the packages work OK

    ```
    $ vagrant up test --provider virtualbox
    $ vagrant ssh test
    ```

    Once inside the vagrant box

    ```
    $ sudo pkgadd -y -d go-agent-VERSION-solaris
    $ sudo pkgadd -y -d go-server-VERSION-solaris
    ```

# To run the GoCD agent or server —

- Ensure that java and readlink are on `PATH`

    ```
    $ sudo pkgutil --upgrade -y jre8 coreutils
    $ sudo ln -sf /opt/csw/java/jre/jre8/bin/java /usr/bin/java
    $ sudo ln -sf /opt/csw/bin/greadlink /usr/bin/readlink
    ```

- Enable the service

    ```
    $ sudo svcs go/agent # check the status
    $ sudo svcadm clear go/agent # to disable maintenance mode
    $ sudo svcadm enable -s go/agent # enable the service
    $ sudo svcadm disable -s go/agent # enable the service
    ```
