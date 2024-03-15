# Design Philosophy of the CC Trusted Launcher


We aim to design the system to protect the workload by ensuring the workload is booted/loaded in a privacy-preserving environment.

And if the workloads are changed on the fly during runtime, it needs to be verified again to ensure the workload is running in a privacy-preserving environment.


This requires the execution environment to be attested and signed with hardware to endorse that it is running in a trusted execution environment.

And to be justified for integrity preserving during its runtime.



The Integrity can be examined from three perspectives:

1.  Preventing unauthorized subjects from making modifications

2.  Preventing authorized subjects from making unauthorized modifications, such as mistakes

3.  Maintaining the internal and external consistency of objects.
    so that their data is a correct and true reflection of the real world and any relationship with any other object
    is valid, consistent, and verifiable

For integrity to be maintained on a system, controls must be in place to restrict access to data, objects, and resources. Maintaining and validating object integrity across storage, transport, and processing requires numerous variations of controls and oversight.
