## Container portability vs application portability

A multi-architecture container image makes it possible to run the same application on different CPU architectures, but this alone does not guarantee a successful migration.

The complete recovery test also required validating:

- persistent volumes
- configuration files
- databases
- file permissions
- exposed ports
- DNS functionality
- application health

The real portability test was not whether the container could start on AMD64, but whether the original application state could be successfully restored and used.