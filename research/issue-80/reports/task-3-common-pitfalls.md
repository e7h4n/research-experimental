# Task 3: Common Pitfalls and Troubleshooting Strategies

## Critical Configuration Pitfalls

### 1. IOMMU Group Isolation Issues

According to [Proxmox documentation](https://pve.proxmox.com/wiki/PCI_Passthrough), improper IOMMU group configuration is a primary failure point:

**Problem**: Devices sharing IOMMU groups cannot be assigned independently  
**Symptoms**: "Failed to assign device: Operation not permitted"

**Solutions**:
- Enable ACS (Access Control Services) in BIOS
- Try different PCIe slots for better isolation
- As a last resort, use ACS override patch (security risk)

**Warning from [Proxmox Forums](https://forum.proxmox.com/threads/pci-gpu-passthrough-on-proxmox-ve-8-installation-and-configuration.130218/)**: 
> "Please don't advise `pcie_acs_override` without pointing out that the VM can then read all of the Proxmox host memory"

### 2. Interrupt Remapping Failures

[Proxmox Wiki](https://pve.proxmox.com/wiki/PCI_Passthrough) states definitively:
> "It is not possible to use PCI passthrough without interrupt remapping"

**Problem**: Missing interrupt remapping support  
**Error Message**: "Interrupt Remapping hardware not found, passing devices to unprivileged domains is insecure"

**Solutions**:
- Verify CPU supports VT-d/AMD-Vi with interrupt remapping
- Update BIOS/UEFI to latest version
- Check kernel parameters include `intel_iommu=on` or `amd_iommu=on`

### 3. BIOS/UEFI Configuration Errors

#### Resizable BAR Issues
According to [technical forums](https://forum.proxmox.com/threads/gpu-passthrough-for-an-amd-system.156444/):
- **Problem**: "Code 43" errors in Windows guests with Resizable BAR enabled
- **Affected GPUs**: AMD Vega and newer, potentially H200
- **Solution**: Disable Resizable BAR/Smart Access Memory in BIOS

#### Primary Display Boot Problems
[GPU passthrough guides](https://github.com/QaidVoid/Complete-Single-GPU-Passthrough) warn:
- **Problem**: Error 43 when booting with passthrough GPU as primary
- **Solution**: Always use integrated graphics or secondary GPU for host

### 4. Driver Binding and Blacklisting Mistakes

Based on [ArchWiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF):

**Common Mistakes**:
- Not blacklisting nouveau/nvidia drivers on host
- Incorrect VFIO-PCI binding timing
- Missing driver modules in initramfs

**Correct Configuration**:
```bash
# /etc/modprobe.d/blacklist.conf
blacklist nouveau
blacklist nvidia
blacklist nvidia_modeset
blacklist nvidia_uvm
blacklist nvidia_drm

# /etc/modprobe.d/vfio.conf
options vfio-pci ids=10de:xxxx,10de:yyyy
softdep nvidia pre: vfio-pci
```

## H200-Specific Pitfalls

### 5. vGPU Software Incompatibility

[NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/nvidia-h200-sxm-vgpu-support-for-vmware-esxi-8-versio/330541) confirm:
- **Problem**: vGPU software not supported on H200 GPUs
- **Impact**: Cannot use traditional vGPU features
- **Workaround**: Use MIG or full passthrough instead

### 6. Memory Capacity Challenges

With H200's 141 GB memory ([Lenovo documentation](https://lenovopress.lenovo.com/lp1944-nvidia-h200-141gb-gpu)):
- **Problem**: BIOS may not support large BAR sizes
- **Symptoms**: GPU not visible or partial memory detection
- **Solutions**: 
  - Update to latest BIOS/UEFI
  - Enable Above 4G Decoding
  - Check for 64-bit BAR support

### 7. NVLink Topology Limitations

[NVIDIA Fabric Manager Guide](https://docs.nvidia.com/datacenter/tesla/fabric-manager-user-guide/index.html) warns:
- **Problem**: Asymmetric bandwidth in 4x GPU configurations
- **Impact**: Performance degradation for certain workloads
- **Mitigation**: Plan VM GPU assignments considering topology

## Common Kernel Parameter Errors

### 8. Invalid or Deprecated Parameters

According to [Proxmox Forums](https://forum.proxmox.com/threads/pci-gpu-passthrough-on-proxmox-ve-8-installation-and-configuration.130218/):

**Incorrect Parameters**:
- `amd_iommu=on` - Actually invalid (on by default)
- `vfio_virqfd` - No longer separate module in kernel 6.2+
- `pci=nomsi` - Can cause stability issues

**Correct GRUB Configuration**:
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```

### 9. MIG Mode Configuration Errors

[NVIDIA MIG Guide](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/index.html) identifies:

**Problem**: NVLinks disabled in MIG mode  
**Impact**: No GPU-to-GPU communication  
**Important Note**: 
> "After the MIG mode is successfully disabled, the GPU NVLinks will be enabled again"

**VM-Specific Issue**:
> "If using MIG inside a VM with passthrough, you may need to reboot the VM as GPU reset is not allowed via hypervisor"

## Performance-Related Pitfalls

### 10. Suboptimal CPU Configuration

[Technical guides](https://gist.github.com/k-amin07/47cb06e4598e0c81f2b42904c6909329) recommend avoiding:
- Default CPU governor settings
- Unpinned vCPUs
- Incorrect NUMA configuration

**Best Practices**:
```bash
# Set performance governor
cpupower frequency-set -g performance

# Pin vCPUs to physical cores
<vcpupin vcpu='0' cpuset='0'/>
<vcpupin vcpu='1' cpuset='1'/>
```

## Troubleshooting Strategies

### Diagnostic Commands

```bash
# Check IOMMU groups
find /sys/kernel/iommu_groups/ -type l

# Verify VFIO binding
lspci -nnk -d 10de:

# Check dmesg for errors
dmesg | grep -E "IOMMU|VFIO|vfio"

# Verify MIG status (if applicable)
nvidia-smi -mig 1
nvidia-smi mig -lgip
```

### Common Error Messages and Solutions

| Error Message | Likely Cause | Solution |
|--------------|--------------|----------|
| "Code 43" in Windows | Multiple causes | Disable Resizable BAR, hide KVM, check ROM |
| "Operation not permitted" | IOMMU group conflict | Check ACS, move PCIe slot |
| "No IOMMU detected" | BIOS setting | Enable VT-d/AMD-Vi |
| "Device busy" | Driver already bound | Blacklist conflicting drivers |
| "Reset bug" | AMD GPU issue | Use vendor-reset module |

## Migration and Operational Pitfalls

### 11. VM Migration Restrictions

[Red Hat documentation](https://docs.redhat.com/en/documentation/red_hat_virtualization/4.3/html/setting_up_an_nvidia_gpu_for_a_virtual_machine_in_red_hat_virtualization/proc_nvidia_gpu_passthrough_nvidia_gpu_passthrough) clearly states:
> "VMs with passed-through devices cannot be migrated"

**Impact**: 
- No live migration capability
- Reduced high availability options
- Maintenance window requirements increase

### 12. Security Implications

[Proxmox Forums](https://forum.proxmox.com/threads/pci-gpu-passthrough-on-proxmox-ve-8-installation-and-configuration.130218/) emphasize:
- ACS override creates security vulnerabilities
- VMs can potentially access host memory
- Isolation guarantees are compromised

## Prevention Checklist

- [ ] BIOS/UEFI updated to latest version
- [ ] IOMMU/VT-d enabled and verified
- [ ] ACS enabled (if supported)
- [ ] Resizable BAR disabled
- [ ] Above 4G Decoding enabled
- [ ] Correct kernel parameters set
- [ ] Drivers properly blacklisted
- [ ] VFIO modules loaded early
- [ ] IOMMU groups verified
- [ ] Primary display not set to passthrough GPU

## References

- [Proxmox PCI Passthrough Wiki](https://pve.proxmox.com/wiki/PCI_Passthrough)
- [NVIDIA MIG User Guide](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/index.html)
- [Arch Linux PCI Passthrough Guide](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)
- [Proxmox GPU Passthrough Tutorial](https://forum.proxmox.com/threads/pci-gpu-passthrough-on-proxmox-ve-8-installation-and-configuration.130218/)
- [NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/nvidia-h200-sxm-vgpu-support-for-vmware-esxi-8-versio/330541)
- [Complete Single GPU Passthrough Guide](https://github.com/QaidVoid/Complete-Single-GPU-Passthrough)