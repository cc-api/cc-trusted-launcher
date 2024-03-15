# Design for Trusted Container Launcher

## Components of Trusted Container Launcher

### 1. kubelet

1. extend `startContainer` method for getting integrity measurement eventlog the start container event.

   The event should have container ID and POD ID associated.  

   `startContainer` is defined in `pkg/kubelet/kuberuntime/kuberuntime_container.go`.

   This method calls the Container Runtime Interface (CRI) to create and start containers.


2. extend `pullImage` method for attestation of the image against golden value.

   The logic for pulling container images is defined in `pkg/kubelet/kuberuntime/kuberuntime_manager.go`.

   The pullImage method encapsulates the details of interacting with the container runtime, utilizing Kubernetes' Container Runtime Interface (CRI) to pull trusted images from trusted source.

  

### 2. containerd

1. extend the `Run` function, so that it returns detailed information about container start commands as a message to kubelet.
    

### 3. launcher 

1. received the integrity measurement eventlog and send it to attestor

2. verify image against its golden value,

    and add this verification cmd into integrity measurement eventlog

    send it along with the golden value to attestor


### 4. attestor

either remote or local, supports two functions:

1. attestation for image/artifacts

2. attestation for eventlogs



### 5. image-storage

1. extend a field in the meta data for golden value, follow SVID (TBD)

2. follow the RA-TLS protocol


### 6. k8s control plane

1. add a new operator as plugin for confidential container specific scheduling 
  
3. add Custom Resource Definitions (CRDs) for defining the confidential container



    

## Workflow

The high-level flow is as follows:

1. The client requests K8S to create a confidential container

2. Kubernetes scheduler decides which node a Pod should run on, based on factors like resource requirements, affinity/anti-affinity, taints, and tolerations.

3. Once a Pod is scheduled to a node, the kubelet on that node is notified that a new Pod needs to be run.

4. If the required container image is not on the node, the kubelet commands the container runtime through the Container Runtime Interface (CRI) to pull the image from the container image registry.

5. After the image is pulled, the CRI invokes the integrity measurement for the measurement of the pulled image.

6. The CRI also invokes image golden value retrieval.

7. Trusted Container Launcher

  - 7.1 In the secure boot mode, the image is attested locally.
  
    - 7.1.1 The image storage not only stores the images but also the image's golden value.
  
    - 7.1.2 The Trusted Container Launcher attests the image's golden value against the measurement, if matches, it proceeds, if not, it returns "NOT verified".

    - 7.1.3 If it matches, the Trusted Container Launcher also triggers the measurement of the container loading process, and sends eventlog to remote attestor.

  - 7.2 In remote attestation mode, Trusted Container Launcher sends the report (image golden value) to the remote attestor. 
      
    - 7.2.1 The remote attestor requests for image storage service for golden value of the image, and attest from here. 
      
    - 7.2.2 If it matches, the Trusted Container Launcher also triggers the measurement of the container loading process, and sends eventlog to remote attestor.

8. After the image is attested (either locally or remotely).

   The container runtime creates the container based on the Pod's specification.

   The containerd lays out the container's filesystem, and creates the necessary config information.

10. The containerd invokes the runtime over an API to create/start/stop the container, of which the create process is measured, and eventlog is sent to remote attestor.

11. The runc is invoked to execute the container.




The following sequence diagram shows the flow of actions for starting a confidential container through the Trusted Container Launcher.

```mermaid
sequenceDiagram
    participant kubelet
    participant containerd
    participant shim
    participant launcher
    participant attestor
    participant image-storage

    autonumber
    kubelet-->>image-storage: Pull image and golden value
    kubelet-->>containerd: Create container
    launcher->launcher: start integrity measurement of event
    Note right of containerd: Save container metadata
    containerd-->>kubelet: Container ID

    kubelet-->>containerd: Create task

    %% Start shim
    containerd-->shim: Prepare bundle
    containerd->>shim: Execute binary: containerd-shim-runc-v2 start
    launcher-->>attestor: send integrity measurement of event
    shim->shim: Start TTRPC server
    shim-->>containerd: Respond with address: unix://containerd/container.sock
    containerd-->>shim: Create TTRPC client

    %% Schedule task

    Note right of containerd: Schedule new task

    containerd->>shim: TaskService.CreateTaskRequest
    launcher->launcher: start integrity measurement of event
    launcher-->>attestor: send integrity measurement of event
    attestor-->image-storage: request image's golden value
    attestor->attestor: verify the image against the golden value
    shim-->>containerd: Task PID
    containerd-->>kubelet: Task ID

    %% Start task

    kubelet->>containerd: Start task

    containerd->>shim: TaskService.StartRequest
    shim-->>containerd: OK
    launcher->>attestor: send measurements

    %% Wait task

    kubelet->>containerd: Wait task

    containerd->>shim: TaskService.WaitRequest
    Note right of shim: Block until task exits
    shim-->>containerd: Exit status

    containerd-->>kubelet: OK

    Note over kubelet,shim: Other task requests (Kill, Pause, Resume, CloseIO, Exec, etc)

    %% Kill signal

    opt Kill task

    kubelet->>containerd: Kill task

    containerd->>shim: TaskService.KillRequest
    shim-->>containerd: OK

    containerd-->>kubelet: OK

    end

    %% Delete task

    kubelet->>containerd: Task Delete

    containerd->>shim: TaskService.DeleteRequest
    shim-->>containerd: Exit information

    containerd->>shim: TaskService.ShutdownRequest
    shim-->>containerd: OK

    containerd-->shim: Close client
    containerd->>shim: Execute binary: containerd-shim-runc-v2 delete
    containerd-->shim: Delete bundle

    containerd-->>kubelet: Exit code
```

