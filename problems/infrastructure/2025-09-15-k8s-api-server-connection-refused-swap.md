# Kubernetes API Server Connection Refused - Swap Memory Issue

**Date:** 2025-09-15  
**Reporter:** Salwan Mohamed  
**Environment:** Production  
**Severity:** Critical  
**Tags:** `kubernetes` `k8s` `swap` `api-server` `infrastructure` `quick-fix`

## 📋 Problem Summary

Kubernetes cluster API server becomes unreachable with "connection refused" errors on port 6443. The kubelet service fails to communicate with the master node API server, causing cluster connectivity issues.

## 🔍 Problem Details

### Symptoms
- API server unreachable on port 6443
- kubectl commands fail with connection refused
- Cluster nodes unable to communicate with master
- Services and pods may become unresponsive

### Environment Information
- **OS:** Linux (Ubuntu/CentOS)
- **Technology Stack:** Kubernetes on-premises cluster
- **Versions:** Kubernetes cluster
- **Configuration:** Multi-master setup with master01 node (IP: 10.1.5.216)

### Error Messages/Logs

```
E0915 09:17:40.623444  101072 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://master01:6443/api?timeout=32s\": dial tcp 10.1.5.216:6443: connect: connection refused"
E0915 09:17:40.626026  101072 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://master01:6443/api?timeout=32s\": dial tcp 10.1.5.216:6443: connect: connection refused"
E0915 09:17:40.627848  101072 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://master01:6443/api?timeout=32s\": dial tcp 10.1.5.216:6443: connect: connection refused"
E0915 09:17:40.629714  101072 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://master01:6443/api?timeout=32s\": dial tcp 10.1.5.216:6443: connect: connection refused"
E0915 09:17:40.631367  101072 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://master01:6443/api?timeout=32s\": dial tcp 10.1.5.216:6443: connect: connection refused"
The connection to the server master01:6443 was refused - did you specify the right host or port?
```

### Screenshots

[Error messages showing repeated connection refused errors to master01:6443]

## 🔧 Investigation Process

### Steps Taken
1. Checked API server status and logs
2. Verified network connectivity to master node
3. Checked kubelet service status
4. Investigated system resource usage
5. Checked for swap memory usage (common k8s issue)

### Findings
- Swap memory was enabled on the master nodes
- Kubernetes requires swap to be disabled for proper operation
- kubelet service fails when swap is detected
- API server becomes unreachable when kubelet fails

## ✅ Solution

### Root Cause
Kubernetes requires swap memory to be disabled on all nodes. When swap is enabled, the kubelet service fails to start properly, causing the API server to become unreachable. This is a known Kubernetes requirement for performance and resource management reasons.

### Fix Applied

```bash
# Run on ALL master nodes (and worker nodes)
# 1. Disable swap immediately
sudo swapoff -a

# 2. Permanently disable swap in fstab
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# 3. Restart kubelet service
sudo systemctl restart kubelet
```

### Step-by-Step Resolution
1. **SSH to each master node** and run the commands above
2. **Verify swap is disabled**: `free -h` should show 0 swap
3. **Check kubelet status**: `sudo systemctl status kubelet`
4. **Test API connectivity**: `kubectl cluster-info`
5. **Repeat on all master and worker nodes**

### Configuration Changes

```bash
# /etc/fstab - swap line commented out
# /swapfile none swap sw 0 0  # <- this line gets commented
```

## 🧪 Testing

### Verification Steps
1. Check swap status: `free -h` (Swap should show 0)
2. Verify kubelet is running: `sudo systemctl status kubelet`
3. Test API server connectivity: `kubectl get nodes`
4. Check cluster status: `kubectl cluster-info`
5. Verify all nodes are Ready: `kubectl get nodes -o wide`

### Test Results
- Swap memory disabled on all nodes ✅
- kubelet service running successfully ✅  
- API server responding on port 6443 ✅
- All cluster nodes in Ready state ✅
- kubectl commands working normally ✅

## 🚨 Prevention

### Monitoring
- Add monitoring for swap usage across all k8s nodes
- Set up alerts for kubelet service failures
- Monitor API server connectivity continuously
- Create health checks for cluster connectivity

### Process Changes
- **Permanently disable swap on system boot** to prevent recurrence after reboots:

```bash
# Run this on ALL master and worker nodes
sudo systemctl mask swap.target
```

### Additional Prevention Steps
```bash
# Add to system initialization scripts
echo 'vm.swappiness=0' >> /etc/sysctl.conf

# Verify swap is permanently disabled
sudo systemctl is-enabled swap.target  # should return "masked"

# Add to server provisioning checklist
# ✓ Disable swap memory
# ✓ Mask swap.target service  
# ✓ Verify kubelet starts properly
```

## 📚 Related Resources

- [Kubernetes Official Documentation - Before you begin](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin)
- [Why Kubernetes requires swap to be disabled](https://github.com/kubernetes/kubernetes/issues/53533)
- [kubelet Configuration Reference](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/)
- [Troubleshooting kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/)

## 📝 Notes

This is a common issue when setting up Kubernetes clusters, especially after system reboots or when swap gets re-enabled. The issue affects all nodes in the cluster but must be fixed on each node individually.

**Critical**: Always run the `sudo systemctl mask swap.target` command to prevent swap from being re-enabled on system reboot. Without this, the problem will recur after every server restart.

**Best Practice**: Include swap disabling in your Kubernetes node provisioning automation/scripts to prevent this issue during initial setup.

---

**Solution Rating:** ⭐⭐⭐⭐⭐ (Quick fix, well-documented Kubernetes requirement)  
**Time to Resolve:** 15 minutes (5 minutes per master node)  
**Follow-up Required:** Yes - Verify swap.target is masked on all nodes for permanent fix