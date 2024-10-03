# Ansible Automation Platform Disconnected Installation Checklist

Installing [Red Hat Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible) (AAP) in environments which are fully disconnected from the internet is fairly straightforward with some planning and preparation.  The following checklist will help prepare you for a virtual machine-based installation of AAP on a disconnected network. Running AAP on an [OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift) cluster in a disconnected environment is also a supported option, but is out of scope for this checklist.

A minimal production architecture for AAP consists of three to five virtual machines: an optional (but preferred) dedicated installation host, a single automation controller, a single private automation hub, a dedicated installer-managed database, and optionally an Event-Driven Ansible (EDA) controller.  This architecture provides the basic requirements for running AAP, while providing the flexibility to scale the platform in the future as needed.

![Simple AAP Architecture Diagram](./images/simple-aap-architecture-dark.png#gh-dark-mode-only)![Simple AAP Architecture Diagram](./images/simple-aap-architecture-light.png#gh-light-mode-only)

## Disconnected Installation Checklist

This checklist covers the high-level steps needed to prepare for a disconnected install.  Links to the relevant documentation in the [Disconnected Installation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_installation_guide/index#disconnected-installation) section of the AAP Installation Guide and the [AAP Hardening Guide](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_hardening_guide/index) are included which can be followed for more detailed information.

### AAP Servers
:white_check_mark: Ensure all AAP servers meet the [system requirements](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_planning_guide/index#platform-system-requirements) for compute and storage, operating system, and system software.  RHEL 9 is the recommended OS.

:white_check_mark: Ensure each AAP server has been added to DNS on the disconnected network.  The FQDN of each server will be used in the installation inventory file.

:white_check_mark: Ensure all AAP servers have access to the RHEL BaseOS and AppStream package repositories.  This may be accomplished in one of the following ways (only one is required):
  1. Each server may be subscribed to a Red Hat Satellite server on the disconnected network which provides the BaseOS and AppStream repos.  This is the preferred method.
  1. Each server may be configured to use a yum repository which [mirrors content](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_installation_guide/index#proc-synchronizing-rpm-repositories-by-using-reposync_disconnected-installation) from the BaseOS and AppStream repos.
  1. Where a Satellite server or yum repo mirror are not available, the repos can be accessed from a [locally mounted RHEL DVD image](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_installation_guide/index#accessing-rpm-repositories-for-locally-mounted-dvd_disconnected-installation).

:white_check_mark: Ensure all AAP servers are *not* configured to use the [EPEL repository](https://docs.fedoraproject.org/en-US/epel/).  EPEL contains versions of python packages which conflict with the requirements for AAP.

:white_check_mark: If the RHEL STIG or similar compliance profile must be applied to the AAP servers, apply all of the controls before attempting to install AAP.  Then review the [STIG considerations](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_hardening_guide/index#con-controller-stig-considerations_hardening-aap) section of the AAP Hardening Guide for guidance on disabling certain compliance controls that are known to be blockers for installation and operation.  Applying security controls to AAP servers after installation can make compliance-related issues more difficult to troubleshoot.

### Network
:white_check_mark: Ensure that the [network ports and protocols](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_planning_guide/index#ref-network-ports-protocols_planning) requirements are met for each AAP server, and that the servers can communicate between themselves.
1. If any network or host-based firewalls exist between any of the AAP servers, ensure the proper network ports are open between the AAP servers.
1. If any cloud instance security groups are applied to any of the AAP servers, ensure the proper security group rules are added to permit required traffic.

### Installation

:white_check_mark: Plan to run the installer from a [dedicated installation host](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_hardening_guide/index#con-install-secure-host_hardening-aap) where possible.  If a dedicated installation host cannot be used, run the installer from the controller.

:white_check_mark: If you have user-provided certificates for the AAP services, copy the certificates and keys to the setup bundle directory on the dedicated installation host (or controller) and add the relevant variables to the [installation inventory](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_hardening_guide/index#proc-install-user-pki_hardening-aap).

:white_check_mark: Follow the steps to [download and install AAP on a disconnected network](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_installation_guide/index#installing-the-aap-setup-bundle_disconnected-installation).  The setup bundle is required for disconnected installation.
1. Sensitive variables in the installation inventory file can be kept in an [encrypted vault file](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html-single/red_hat_ansible_automation_platform_hardening_guide/index#ref-sensitive-variables-install-inventory_hardening-aap) to limit exposure.
1. If passwordless sudo is not configured in the environment, the sudo password can be [passed to the installer](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html/red_hat_ansible_automation_platform_hardening_guide/hardening-aap#ref-sudo-nopasswd_hardening-aap) at runtime.
