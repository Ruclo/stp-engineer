# Openshift-virtualization-tests Test plan

## **VM Video Configuration Field - Quality Engineering Plan**

### **Metadata & Tracking**

| Field                  | Details                                                                |
|:-----------------------|:-----------------------------------------------------------------------|
| **Enhancement(s)**     | [VEP #35: Add a Video configuration field for VMs](https://github.com/kubevirt/enhancements/blob/main/veps/sig-compute/vm_video_configuration_field.md) |
| **Feature in Jira**    | CNV-00001                                                              |
| **Jira Tracking**      | CNV-00001 (Tasks must be created to **block the epic**)                |
| **QE Owner(s)**        | ozj                                                                    |
| **Owning SIG**         | sig-compute                                                            |
| **Participating SIGs** | sig-compute                                                            |
| **Current Status**     | Draft                                                                  |

---

## **I. Motivation and Requirements Review**

### **1. Requirement & User Story Review Checklist**

| Review Item                                    | Status | Notes                                           |
|:-----------------------------------------------|:-------|:------------------------------------------------|
| All user stories are clear and testable        | ✅     | Four clear user stories defined in VEP          |
| Acceptance criteria defined for each story     | ✅     | Covered in design and API specification         |
| Non-functional requirements identified         | ✅     | Performance, compatibility, stability NFRs      |
| Edge cases and error scenarios documented      | ✅     | Invalid configurations, arch limitations        |
| Dependencies on other features identified      | ✅     | Architecture, firmware dependencies clear       |

### **2. Technology and Design Review**

| Technology Area              | Design Detail                                          | Testing Implication                                    |
|:-----------------------------|:-------------------------------------------------------|:-------------------------------------------------------|
| **API Changes**              | New `spec.template.spec.domain.devices.video.type`    | Test YAML validation, API compatibility                |
| **Video Device Types**       | virtio, bochs, vga, ramfb                              | Test each type across architectures                    |
| **Architecture Support**     | AMD64 (BIOS/EFI), ARM64, s390x                         | Matrix testing across all platforms                    |
| **Default Behavior**         | AMD64-BIOS: vga, AMD64-EFI: bochs, ARM64/s390x: virtio | Verify defaults when video field omitted               |
| **Validation Logic**         | Architecture-specific video type validation            | Test invalid combinations, error messages              |
| **Backward Compatibility**   | Optional field, no impact on existing VMs              | Verify existing VMs unaffected                         |

---

## **II. Software Test Plan (STP)**

### **1. Introduction and Scope**

This Software Test Plan (STP) covers the quality engineering activities for the **VM Video Configuration Field** feature in OpenShift Virtualization 4.21. The feature introduces an explicit API field allowing users to configure the video device type for virtual machines, providing granular control over graphics capabilities across different architectures.

**Scope includes:**
- API field validation and behavior
- Video device type functionality (virtio, bochs, vga, ramfb)
- Multi-architecture support (AMD64, ARM64, s390x)
- Firmware compatibility (BIOS, UEFI/EFI)
- Guest OS compatibility (Windows, Linux variants)
- Default behavior preservation
- Upgrade and migration scenarios

**Out of scope:**
- Performance benchmarking of video rendering within guest OS
- Third-party video driver development
- Features beyond basic video device configuration

### **2. Test Objectives**

1. Verify that users can successfully configure video device types via the `video.type` API field
2. Validate that all supported video device types function correctly on their respective architectures
3. Ensure proper validation and error handling for invalid video type/architecture combinations
4. Confirm backward compatibility: existing VMs without the video field continue to work with default behavior
5. Verify guest OS compatibility across major operating systems (RHEL, Windows, Ubuntu, Fedora)
6. Validate default video device selection per architecture/firmware combination
7. Ensure no regression in VM boot time or performance with video configuration
8. Verify upgrade scenarios from versions without the video field

### **3. Motivation**

#### **A. User Stories (Testing Perspective)**

| Story # | User Story | Testing Focus |
|:--------|:-----------|:--------------|
| US-1 | As a VM user, I want high-resolution display access via remote display methods | Test virtio video device enables high-resolution displays in VNC/SPICE sessions |
| US-2 | As a developer, I need to install Windows 11 on ARM with proper video support | Test Windows 11 ARM64 installation workflow with appropriate video device configuration |
| US-3 | As a user with legacy guests, I need non-virtio video devices for guests without virtio drivers | Test vga/bochs video devices work with legacy OS (Windows XP, older Linux distributions) |
| US-4 | As a performance-focused user, I want to optimize video performance with virtio drivers | Test virtio video device performance on modern guests with virtio support |

#### **B. Testing Goals**

1. **Functional Completeness:** Verify all video device types work as specified across supported architectures
2. **API Validation:** Ensure the video.type field accepts valid values and rejects invalid configurations
3. **Platform Coverage:** Test across AMD64 (BIOS/EFI), ARM64, and s390x architectures
4. **Guest OS Compatibility:** Validate major guest operating systems function correctly with each video type
5. **Default Behavior:** Confirm correct default video device selection when field is omitted
6. **Error Handling:** Verify clear, actionable error messages for invalid configurations
7. **Backward Compatibility:** Ensure no impact to existing VMs or workflows
8. **Upgrade/Migration:** Verify smooth upgrades from versions without the video field

#### **C. Non-Goals (Testing Scope Exclusions)**

1. Performance benchmarking of video rendering within guest OS applications
2. Testing third-party GPU passthrough or SR-IOV configurations
3. Guest OS-specific video driver installation or troubleshooting
4. Video device hot-plug/hot-unplug scenarios
5. Multi-head display configurations
6. 3D acceleration or GPU-specific features beyond basic video device configuration
7. Video device bandwidth or network streaming performance

### **4. Test Strategy**

#### **A. Types of Testing**

| Test Type | Description | Coverage |
|:----------|:------------|:---------|
| **Functional Testing** | Verify video device types work as specified | All video types, all architectures |
| **API Testing** | Validate YAML/JSON API field behavior | Valid/invalid configurations |
| **Integration Testing** | Test video with firmware, architecture combinations | BIOS/EFI, AMD64/ARM64/s390x |
| **Compatibility Testing** | Guest OS compatibility validation | RHEL, Windows, Ubuntu, Fedora |
| **Negative Testing** | Invalid configurations, error handling | Unsupported type/arch combinations |
| **Regression Testing** | Existing VM behavior unchanged | VMs without video field |
| **Upgrade Testing** | Upgrade from previous versions | 4.20 → 4.21 upgrade scenarios |
| **Performance Testing** | VM boot time impact measurement | Baseline vs. video-configured VMs |

#### **B. Potential Areas to Consider**

1. **Architecture-specific behavior:**
   - AMD64 with BIOS firmware
   - AMD64 with UEFI/EFI firmware
   - ARM64 (virtio default)
   - s390x (virtio default)

2. **Video device type variations:**
   - virtio (modern, high-performance)
   - bochs (common for EFI)
   - vga (legacy compatibility)
   - ramfb (firmware framebuffer)

3. **Guest OS matrix:**
   - RHEL 8.x, 9.x
   - Windows 10, Windows 11, Windows Server
   - Ubuntu 20.04, 22.04, 24.04
   - Fedora (latest)

4. **VM lifecycle operations:**
   - VM creation with video configuration
   - VM start/stop/restart
   - VM migration (live and cold)
   - VM snapshot/restore

5. **API validation scenarios:**
   - Omitted video field (default behavior)
   - Valid video type for architecture
   - Invalid video type for architecture
   - Malformed YAML/JSON

6. **Remote display protocols:**
   - VNC console access
   - SPICE console (if applicable)
   - Resolution changes

### **5. Test Environment**

| Environment Component | Specification |
|:----------------------|:--------------|
| **OpenShift Version** | 4.21 (target release) |
| **CNV Version** | 4.21 |
| **Architectures** | AMD64 (BIOS and UEFI), ARM64, s390x |
| **Cluster Size** | Multi-node cluster (3+ worker nodes) |
| **Storage** | CSI-compatible storage for VM disks |
| **Guest OS Images** | RHEL 8.10, RHEL 9.4, Windows 10, Windows 11, Ubuntu 22.04, Fedora 40 |
| **Network** | Pod network + multus (if testing secondary networks) |
| **Remote Access Tools** | VNC client, virtctl console |

### **6. Entry and Exit Criteria**

#### **A. Entry Criteria**

- [ ] Feature code merged to main branch
- [ ] API documentation updated with video field specification
- [ ] Feature available in target OpenShift Virtualization 4.21 build
- [ ] Test environment provisioned with all required architectures
- [ ] Guest OS images available for testing
- [ ] Test cases documented and reviewed

#### **B. Exit Criteria**

- [ ] All P1 test scenarios executed and passed
- [ ] 100% P1 test pass rate
- [ ] ≥ 95% P2 test pass rate
- [ ] All critical/blocker bugs resolved
- [ ] No open high-severity bugs
- [ ] Performance metrics meet NFR criteria (< 5% boot time impact)
- [ ] Documentation validated (user guide, API reference)
- [ ] Upgrade testing completed successfully
- [ ] QE sign-off obtained

### **7. Risk Management**

| Risk | Impact | Probability | Mitigation Strategy |
|:-----|:-------|:------------|:--------------------|
| **Limited ARM64 test hardware availability** | High | Medium | Prioritize ARM64 testing early, use cloud resources if needed |
| **Windows guest licensing constraints** | Medium | Low | Coordinate with team for Windows test licenses |
| **Architecture-specific bugs difficult to reproduce** | High | Medium | Maintain dedicated test environments per architecture |
| **Guest OS compatibility issues** | Medium | Medium | Test with multiple OS versions, document known limitations |
| **Performance regression not caught** | Medium | Low | Implement automated boot time benchmarking |
| **Upgrade scenario edge cases** | High | Medium | Test multiple upgrade paths (4.19→4.21, 4.20→4.21) |
| **Invalid configuration causing VM failures** | High | Low | Comprehensive negative testing, validation logic review |

### **8. Limitations**

1. **Architecture Availability:** Testing on s390x may be limited by hardware availability
2. **Guest OS Coverage:** Cannot test every possible Linux distribution or Windows version
3. **Video Performance Metrics:** Subjective display quality not quantitatively measured
4. **Third-party Tools:** VNC client compatibility variations not exhaustively tested
5. **Scale Testing:** Large-scale concurrent VM deployment with varied video configs may be limited by cluster resources
6. **Legacy OS Support:** Extremely old operating systems (e.g., Windows XP) tested on best-effort basis

---

## **III. Test Case Descriptions & Traceability**

### **1. Test Case Repository**

Test cases will be maintained in the OpenShift Virtualization test repository:
- **Location:** `openshift-virtualization-tests/tests/compute/video_configuration/`
- **Automation Framework:** Ginkgo/Gomega
- **Manual Test Cases:** Documented in Polarion

### **2. Traceability Matrix**

| Requirement | User Story | Test Scenario ID(s) | Priority |
|:------------|:-----------|:--------------------|:---------|
| Configure video.type field | US-1, US-4 | TC-VIDEO-001, TC-VIDEO-002 | P1 |
| Default video device per arch | All | TC-VIDEO-003, TC-VIDEO-004, TC-VIDEO-005 | P1 |
| Virtio video device | US-1, US-4 | TC-VIDEO-006, TC-VIDEO-007 | P1 |
| VGA video device | US-3 | TC-VIDEO-008 | P1 |
| Bochs video device | US-2 | TC-VIDEO-009 | P1 |
| Ramfb video device | US-2 | TC-VIDEO-010 | P2 |
| Windows guest compatibility | US-2 | TC-VIDEO-011, TC-VIDEO-012 | P1 |
| Linux guest compatibility | US-1, US-3, US-4 | TC-VIDEO-013, TC-VIDEO-014 | P1 |
| Invalid configuration validation | N/A | TC-VIDEO-015, TC-VIDEO-016 | P1 |
| Backward compatibility | N/A | TC-VIDEO-017 | P1 |
| VM migration with video config | N/A | TC-VIDEO-018 | P2 |
| Upgrade scenario | N/A | TC-VIDEO-019 | P2 |
| Performance impact | NFR-1 | TC-VIDEO-020 | P2 |

### **3. Test Scenario Descriptions**

#### **High Priority (P1) Scenarios**

**TC-VIDEO-001: Create VM with explicit virtio video type**
- **Precondition:** OpenShift Virtualization 4.21 installed
- **Steps:**
  1. Create VM YAML with `spec.template.spec.domain.devices.video.type: virtio`
  2. Apply the VM configuration
  3. Start the VM
  4. Verify VM starts successfully
  5. Connect via VNC and verify display is functional
- **Expected Result:** VM starts successfully with virtio video device
- **Automation:** Yes

**TC-VIDEO-002: Create VM with vga video type on AMD64-BIOS**
- **Precondition:** AMD64 cluster with BIOS firmware support
- **Steps:**
  1. Create VM YAML with `spec.template.spec.domain.devices.video.type: vga`
  2. Set firmware to BIOS
  3. Apply and start the VM
  4. Verify VM starts successfully
- **Expected Result:** VM starts with vga video device
- **Automation:** Yes

**TC-VIDEO-003: Verify default video device on AMD64-BIOS (no video field)**
- **Precondition:** AMD64 cluster
- **Steps:**
  1. Create VM YAML without video field
  2. Set firmware to BIOS
  3. Start VM and inspect video device in guest
- **Expected Result:** VM uses vga video device by default
- **Automation:** Yes

**TC-VIDEO-004: Verify default video device on AMD64-EFI (no video field)**
- **Precondition:** AMD64 cluster with UEFI support
- **Steps:**
  1. Create VM YAML without video field
  2. Set firmware to UEFI/EFI
  3. Start VM and inspect video device in guest
- **Expected Result:** VM uses bochs video device by default
- **Automation:** Yes

**TC-VIDEO-005: Verify default video device on ARM64 (no video field)**
- **Precondition:** ARM64 cluster
- **Steps:**
  1. Create VM YAML without video field
  2. Start VM and inspect video device in guest
- **Expected Result:** VM uses virtio video device by default
- **Automation:** Yes

**TC-VIDEO-006: Virtio video device enables high resolution display**
- **Precondition:** VM with virtio video configured
- **Steps:**
  1. Start VM with virtio video device
  2. Connect via VNC
  3. Attempt to set resolution to 1920x1080 or higher in guest OS
- **Expected Result:** High resolution is supported and display renders correctly
- **Automation:** Partial (manual verification of display quality)

**TC-VIDEO-007: Virtio video device on supported architectures**
- **Precondition:** AMD64, ARM64, s390x clusters available
- **Steps:**
  1. Create VM with virtio video on each architecture
  2. Start VMs and verify successful boot
- **Expected Result:** Virtio works on all supported architectures
- **Automation:** Yes

**TC-VIDEO-008: VGA video device for legacy guest OS**
- **Precondition:** AMD64 cluster, legacy OS image (e.g., Windows 7, older Linux)
- **Steps:**
  1. Create VM with vga video device
  2. Install/boot legacy OS
  3. Verify display is functional without virtio drivers
- **Expected Result:** VGA provides basic display for legacy guests
- **Automation:** Partial (requires legacy OS image)

**TC-VIDEO-009: Bochs video device on AMD64-EFI**
- **Precondition:** AMD64 cluster with UEFI firmware
- **Steps:**
  1. Create VM with bochs video device
  2. Set firmware to UEFI
  3. Start VM and verify boot
- **Expected Result:** VM boots successfully with bochs video
- **Automation:** Yes

**TC-VIDEO-011: Windows 10 guest compatibility with virtio video**
- **Precondition:** Windows 10 guest image, virtio drivers available
- **Steps:**
  1. Create VM with virtio video device
  2. Start Windows 10 VM
  3. Install virtio drivers if needed
  4. Verify display functionality
- **Expected Result:** Windows 10 works correctly with virtio video
- **Automation:** Partial (requires Windows license)

**TC-VIDEO-012: Windows 11 ARM64 installation with video config**
- **Precondition:** ARM64 cluster, Windows 11 ARM ISO
- **Steps:**
  1. Create VM with appropriate video device for ARM64
  2. Start Windows 11 ARM installation
  3. Complete installation process
  4. Verify display throughout installation and post-install
- **Expected Result:** Windows 11 ARM installs successfully
- **Automation:** Manual (complex installation flow)

**TC-VIDEO-013: RHEL 9 guest compatibility with virtio video**
- **Precondition:** RHEL 9 guest image
- **Steps:**
  1. Create VM with virtio video device
  2. Start RHEL 9 VM
  3. Verify display functionality
  4. Test VNC console access
- **Expected Result:** RHEL 9 works correctly with virtio video
- **Automation:** Yes

**TC-VIDEO-014: Ubuntu 22.04 guest compatibility with virtio video**
- **Precondition:** Ubuntu 22.04 guest image
- **Steps:**
  1. Create VM with virtio video device
  2. Start Ubuntu VM
  3. Verify display and console access
- **Expected Result:** Ubuntu works correctly with virtio video
- **Automation:** Yes

**TC-VIDEO-015: Invalid video type for architecture rejected**
- **Precondition:** Knowledge of unsupported video/arch combinations
- **Steps:**
  1. Create VM YAML with invalid video type for target architecture
  2. Attempt to apply configuration
- **Expected Result:** API validation rejects the configuration with clear error message
- **Automation:** Yes

**TC-VIDEO-016: Malformed video field rejected**
- **Precondition:** N/A
- **Steps:**
  1. Create VM YAML with invalid video type value (e.g., "invalid-type")
  2. Attempt to apply configuration
- **Expected Result:** API validation rejects with clear error message
- **Automation:** Yes

**TC-VIDEO-017: Existing VMs without video field unaffected (backward compatibility)**
- **Precondition:** Existing VM created in version < 4.21
- **Steps:**
  1. Upgrade cluster to 4.21
  2. Start existing VM without video field
  3. Verify VM behavior unchanged
- **Expected Result:** Existing VMs continue to work with default video devices
- **Automation:** Yes

#### **Medium Priority (P2) Scenarios**

**TC-VIDEO-010: Ramfb video device on ARM64**
- **Precondition:** ARM64 cluster
- **Steps:**
  1. Create VM with ramfb video device
  2. Start VM and verify boot
- **Expected Result:** VM boots with ramfb framebuffer
- **Automation:** Yes

**TC-VIDEO-018: Live migration with video configuration**
- **Precondition:** Multi-node cluster, VM with video.type configured
- **Steps:**
  1. Start VM with specific video configuration
  2. Initiate live migration to another node
  3. Verify migration completes successfully
  4. Verify video configuration persists and VM display remains functional
- **Expected Result:** Migration succeeds, video config unchanged
- **Automation:** Yes

**TC-VIDEO-019: Upgrade from 4.20 to 4.21 with video config**
- **Precondition:** Cluster running 4.20
- **Steps:**
  1. Create VMs in 4.20 (without video field)
  2. Upgrade cluster to 4.21
  3. Verify existing VMs continue to function
  4. Create new VM with video.type field
  5. Verify new field works as expected
- **Expected Result:** Smooth upgrade, existing VMs unaffected, new field functional
- **Automation:** Partial

**TC-VIDEO-020: Performance impact of video configuration**
- **Precondition:** Identical VMs with and without video configuration
- **Steps:**
  1. Measure boot time of VM without video field (baseline)
  2. Measure boot time of VM with video.type configured
  3. Compare boot times
- **Expected Result:** Boot time increase < 5% (NFR-1)
- **Automation:** Yes

### **4. Traceability Maintenance**

- Test case IDs will be maintained in Polarion and linked to Jira epic CNV-00001
- Each test execution will be tracked in Polarion test runs
- Automation coverage will be tracked via CI/CD pipeline reporting
- Traceability matrix will be updated when new requirements or edge cases are discovered

---

## **IV. Sign-off and Approval**

### **1. Final Sign-off Checklist**

- [ ] All P1 test cases executed with 100% pass rate
- [ ] All P2 test cases executed with ≥ 95% pass rate
- [ ] No critical or blocker bugs open
- [ ] Performance NFRs validated and met
- [ ] Backward compatibility confirmed
- [ ] Upgrade testing completed successfully
- [ ] Documentation reviewed and accurate
- [ ] Automation coverage meets target (≥ 80% for P1 scenarios)
- [ ] Risk mitigation strategies executed
- [ ] Test results published and reviewed

### **2. Approval**

| Role | Name | Status | Date |
|:-----|:-----|:-------|:-----|
| **QE Owner** | ozj | Pending | TBD |
| **QE Lead** | TBD | Pending | TBD |
| **Engineering Lead** | TBD | Pending | TBD |
| **Product Owner** | TBD | Pending | TBD |

---

**Document Version:** 1.0
**Last Updated:** 2025-11-12
**Next Review Date:** TBD
