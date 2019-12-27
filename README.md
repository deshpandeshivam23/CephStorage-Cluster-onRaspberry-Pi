# Ceph-Storage-Cluster-On-RBPI

What is CEPH Storage ?

Ceph is open source software defined storage to provide highly scalable object-, block- and file-based storage under a unified system.

Ceph storage clusters are designed to run on commodity hardware, using an algorithm called CRUSH (Controlled Replication Under Scalable Hashing) to ensure data is evenly distributed across the cluster and that all cluster nodes can retrieve data quickly without any centralized bottlenecks

Ceph object storage is accessible through Amazon Simple Storage Service (S3) and OpenStack Swift Representational State Transfer (REST)-based application programming interfaces (APIs), and a native API for integration with software applications.

Ceph block storage makes use of a Ceph Block Device, which is a virtual disk that can be attached to bare-metal Linux-based servers or virtual machines. The Ceph Reliable Autonomic Distributed Object Store (RADOS) provides block storage capabilities, such as snapshots and replication. The Ceph RADOS Block Device is integrated to work as a back end with OpenStack Block Storage.

Ceph file storage makes use of the Portable Operating System Interface (POSIX)-compliant Ceph file system (CephFS) to store data in a Ceph Storage Cluster. CephFS uses the same clustered system as Ceph block storage and Ceph object storage.

    Reference : https://docs.ceph.com/docs/master/
    

#################################################################################
# Build CEPH-on-RBPI Steps :
#################################################################################

Step 1: Compile CEPH On RBPI Using Clang/LLVM
    
        Reference: https://louwrentius.com/compiling-ceph-on-the-raspberry-pi-3b-armhf-using-clangllvm.html
                      
Step 2: Build Cluster
 
        Accordind to Step 1 we consider that CEPH Complie for RBPI
        
        LAB Setup :
        3 * RBPI With Ceph Compile.
        
        The RBPI in this tutorial will use the following hostnames and IP addresses.
        
        hostname            IP address

          ceph-admon              192.168.15.10    # This Node is Used for admin and monitor
          Ceph-osd1               192.168.15.11    # This Node is Used for Object-Storage-Daemon 1 or Manager
          ceph-osd2               192.168.15.12    # This Node is Used for Object-Storage-Daemon 2
          
        
        Note: Manager node is requried for Luminious Platform. Please Refer CEPH Documentations.
        Note : This is for Simple Project Purpose Configuration. Because less Hardware 
        Reference : https://docs.ceph.com/docs/master/
        
        
Step 3 : Configure All Nodes
            
            Create Ceph User for all nodes and give them root privileges
      
      # useradd -d /home/cephuser -m cephuser
      # passwd cephuser
      
      # echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
      # chmod 0440 /etc/sudoers.d/cephuser
      # sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers
      
     Configure Hosts File
     
     # vi /etc/hosts
     
          192.168.15.10   ceph-admon
          192.168.15.11   ceph-osd1
          192.168.15.12   ceph-osd2
          
          
        
