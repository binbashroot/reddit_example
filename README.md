```yaml

---
- hosts: servera.example.com
  become: true
  gather_facts: true
  vars:
   cockpit_addl_packages:
     - cockpit-ws
     - cockpit-system
     - cockpit-storaged
     - cockpit-selinux

   cockpit_package:
     name: cockpit
     weak_deps: false

  tasks:
   - name: "Gather package facts"
     ansible.builtin.package_facts:
       manager: "rpm"

   - set_fact:
       cockpit_optional_pcp: "{{ 'cockpit-pcp' if ansible_facts['packages']['pcp'] is defined }}"
       cockpit_optional_machines: "{{ 'cockpit-machines' if ansible_facts['packages']['libvirt-client'] is defined }}"
       cockpit_optional_podman: "{{ 'cockpit-podman' if ansible_facts['packages']['podman'] is defined }}"

   - name: Append to cockpit_addl_packages list 
     set_fact:
       cockpit_addl_packages: "{{ cockpit_addl_packages + [ item ] }}"
     loop:
       - "{{ cockpit_optional_pcp }}" 
       - "{{ cockpit_optional_machines }}"
       - "{{ cockpit_optional_podman }}"
     when: item|length > 0

   - name: "Install cockpit package"
     ansible.builtin.dnf:
       name: "{{ cockpit_package['name'] }}"
       state: latest
       install_weak_deps: "{{ cockpit_package['weak_deps'] | default(omit) }}"
     become: true

   - name: "Install additional or optional cockpit packages"
     ansible.builtin.dnf:
       name: "{{ cockpit_addl_packages }}"
       state: latest
     become: true
     register: out
```
