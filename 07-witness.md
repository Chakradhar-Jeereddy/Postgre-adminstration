![witness!](witness.jpg)

- The job of the witness server is to prevent split brain syndrom
- Although primary is active, the standby promots itself to primary when it can't communicate to primary due to network issue.
- To avoid this we configure witness server which will be in same network as production and monitors the prod status.
- IT will inform standby not to promote itself when primary is active.
