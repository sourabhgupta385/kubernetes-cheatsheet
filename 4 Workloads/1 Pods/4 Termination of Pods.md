# Termination of Pods

- Because Pods represent running processes on nodes in the cluster, it is important to allow those processes to gracefully terminate when they are no longer needed (vs being violently killed with a KILL signal and having no chance to clean up). 
- Users should be able to request deletion and know when processes terminate, but also be able to ensure that deletes eventually complete. 
- When a user requests deletion of a Pod, the system records the intended grace period before the Pod is allowed to be forcefully killed, and a TERM signal is sent to the main process in each container. 
- Once the grace period has expired, the KILL signal is sent to those processes, and the Pod is then deleted from the API server. 
- If the Kubelet or the container manager is restarted while waiting for processes to terminate, the termination will be retried with the full grace period.

An example flow:

1. User sends command to delete Pod, with default grace period (30s)
2. The Pod in the API server is updated with the time beyond which the Pod is considered “dead” along with the grace period.
3. Pod shows up as “Terminating” when listed in client commands
4. (simultaneous with 3) When the Kubelet sees that a Pod has been marked as terminating because the time in 2 has been set, it begins the Pod shutdown process.
    - If one of the Pod’s containers has defined a `preStop` hook, it is invoked inside of the container. 
    - If the `preStop` hook is still running after the grace period expires, step 2 is then invoked with a small (2 second) one-time extended grace period. 
    - You must modify `terminationGracePeriodSeconds` if the preStop hook needs longer to complete.
    - The container is sent the TERM signal. Note that not all containers in the Pod will receive the TERM signal at the same time and may each require a preStop hook if the order in which they shut down matters.
5. (simultaneous with 3) Pod is removed from endpoints list for service, and are no longer considered part of the set of running Pods for replication controllers. Pods that shutdown slowly cannot continue to serve traffic as load balancers (like the service proxy) remove them from their rotations.
6. When the grace period expires, any processes still running in the Pod are killed with SIGKILL.
7. The Kubelet will finish deleting the Pod on the API server by setting grace period 0 (immediate deletion).
8. The Pod disappears from the API and is no longer visible from the client.

By default, all deletes are graceful within 30 seconds. The kubectl delete command supports the `--grace-period=<seconds>` option which allows a user to override the default and specify their own value. 
The value 0 force deletes the Pod. You must specify an additional flag `--force` along with `--grace-period=0` in order to perform force deletions.