wiremock-loader
===============

Wiremock-Loader is designed for environments where a wiremock service is
available, but cannot be initialized via the filesystem. (Gitlab CI Services)

The intention is to read the `mappings` / `__files` from the local filesystem,
and inject the stub definitions via the `__admin` interface.

Usage
-----

The system is intended to be run from a system that has yarn / node installed,
such as the `srsbiznas/sbt_yarn_ci` docker image.

    yarn start -m /path/to/mocks -h mock-host -p 12345

> Note that the default value for the hostname is `localhost` and the default
> port is 8080.
