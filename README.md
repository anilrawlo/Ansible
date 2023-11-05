

---
- name: Copy and Unarchive a tar file to a destination directory
  hosts: destination_server  # Replace with your destination server's hostname or IP
  become: yes  # If you need elevated privileges for copying and unarchiving

  vars:
    source_server: source_server  # Replace with your source server's hostname or IP
    QC_number: "123"  # Replace with your desired QC number
    destination_directory: "/tmp/QC{{ QC_number }}/"
    tar_file_name: "your_tar_file_QC{{ QC_number }}.tar"

  tasks:
    - name: Copy the tar file from source server to destination server
      ansible.builtin.copy:
        src: "/path/to/your/tar_files/{{ tar_file_name }}"
        dest: "{{ destination_directory }}{{ tar_file_name }}"
      delegate_to: "{{ source_server }}"

    - name: Create the destination directory on the destination server
      file:
        path: "{{ destination_directory }}"
        state: directory

    - name: Unarchive the tar file on the destination server
      ansible.builtin.unarchive:
        src: "{{ destination_directory }}{{ tar_file_name }}"
        dest: "{{ destination_directory }}"
        remote_src: yes
        creates: "{{ destination_directory }}"


      =================


      ---
- name: Copy files to multiple destination paths with CSV
  hosts: localhost  # You can specify your target hosts here
  become: yes  # If you need elevated privileges for copying files

  tasks:
    - name: Read file entries from CSV file
      ansible.builtin.read_csv:
        path: file_entries.csv
        delimiter: ","
      register: file_entries_content

    - name: Split and create a list of file entries
      set_fact:
        file_entries_list: "{{ file_entries_content.list }}"
      changed_when: false

    - name: Take a backup of existing files
      copy:
        src: "{{ item.destination }}{{ item.filename }}"
        dest: "/tmp/QC{{ qc_number }}/{{ item.filename }}.bak"
      with_items: "{{ file_entries_list }}"
      when: ansible.builtin.stat(path="{{ item.destination }}{{ item.filename }}").exists

    - name: Copy new files to destinations
      copy:
        src: "{{ item.filename }}"
        dest: "{{ item.destination }}"
        mode: "{{ item.permission }}"
      with_items: "{{ file_entries_list }}"

    - name: Rollback files to original destination
      copy:
        src: "/tmp/QC{{ qc_number }}/{{ item.filename }}.bak"
        dest: "{{ item.destination }}"
        mode: "{{ item.permission }}"
      with_items: "{{ file_entries_list }}"
      when: ansible.builtin.stat(path="/tmp/QC{{ qc_number }}/{{ item.filename }}.bak").exists

