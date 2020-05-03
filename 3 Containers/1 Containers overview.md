# Containers overview

Containers are a technnology for packaging the (compiled) code for an application along with the dependencies it needs at run time. Each container that you run is repeatable; the standardisation from having dependencies included means that you get the same behavior wherever you run it.

- Containers decouple applications from underlying host infrastructure. This makes deployment easier in different cloud or OS environments.
- By design, a container is immutable: you cannot change the code of a container that is already running. 
- If you have a containerized application and want to make changes, you need to build a new container that includes the change, then recreate the container to start from the updated image.