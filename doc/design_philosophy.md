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


Confidentiality and integrity depend on each other. Without object integrity (in other words, the inability of an object to be modified without permission), confidentiality cannot be maintained.


Integrity is dependent on confidentiality and access control. Concepts, conditions, and aspects of integrity include the following:
 
 - Accuracy: Being correct and precise

 - Truthfulness: Being a true reflection of reality

 - Validity: Being factually or logically sound

 - Accountability: Being responsible or obligated for actions and results

 - Responsibility: Being in charge or having control over something or someone Completeness: Having all necessary components or parts

 - Comprehensiveness: Being complete in scope; the full inclusion of all needed elements


This work is mainly focused on using integrity measurement of a confidential computing execution environment. 
