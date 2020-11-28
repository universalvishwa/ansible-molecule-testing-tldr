# TL;DR Creating Ansible roles with Molecule Testing

1. Create Virtual environment and install Python dependencies
    ```bash
    $ python3 -m venv venv
    $ source venv/bin/activate
    $ pip3 install -r requirements.txt 
    ```
2. Create Ansible role using Molecule 
    ```bash
    $ molecule init role <role_name> --driver-name docker
    ```
3. Update the _metadata_ for Ansible role
    - Complete the `meta/main.yml` with appropriate details. **OR**
    - Delete this folder completely to avoid linting errors.
4. Configure `molecule.yml` configuration file
    - _**dependency**_: Add dependent collections/roles to `molecule/default/requirements.yml`
    - _**driver**_: Set the driver for infrastructure to `docker`
    - _**lint**_: Configure linter script (`yamllint`, `ansible-lint`) and place the rule override files in root path of role.
    - _**platforms**_: Set the Dockerized OS distribution platform(s) to run Molecule test
    - _**verifier**_: Set verification provider use to validate the test environment. `ansible`(default), `testinfra`
    - Example of a `molecule.yml` file,
        ```yaml
        ---
        dependency:
          name: galaxy
          options:
            ignore-certs: true
            requirements-file: requirements.yml
            role-file: requirements.yml
        driver:
          name: docker
        lint: |
          set -e
          yamllint .
          ansible-lint
        platforms:
          - name: centos8
            image: geerlingguy/docker-centos8-ansible:latest
            command: ""
            volumes:
              - /sys/fs/cgroup:/sys/fs/cgroup:ro
            privileged: true
            pre_build_image: true
          - name: ubuntu2004
            image: "geerlingguy/docker-ubuntu2004-ansible:latest"
            command: ""
            volumes:
              - /sys/fs/cgroup:/sys/fs/cgroup:ro
            privileged: true
            pre_build_image: true
          - name: debian10
            image: geerlingguy/docker-debian10-ansible:latest
            command: ""
            volumes:
              - /sys/fs/cgroup:/sys/fs/cgroup:ro
            privileged: true
            pre_build_image: true
        provisioner:
          name: ansible
        verifier:
          name: ansible
        ```
4. Configure `converge.yml`
    - Add `pre_tasks` and `vars` (static variables) to converge playbook.
5. Prsoceed with writing code for the Ansible role
    - Populate `tasks/`, `handlers/`, `vars/`, `defaults/` etc...
6. Create Molecule Test instance and run playbook
    ```bash
    $ cd <role_name>
    $ molecule converge
    ```
7. Add Ansible Tasks, playbooks or Testinfra script to `verify.yml` to validate the Molecule test instance.
    - This can be individual tasks or a separate playbook. **OR**
    - Testinfra script
8. Run the validation tests on the Molecule instance to verify the code
    ```bash
    $ cd <role_name>
    $ molecule verify
    ```
9. Follow this cycle process iteratively by adding new code to the role followed by running `molecule converge` and `molecule verify` until the code for the role is code complete.
    - If there is a need to recreate the Molecule testing during development phase use `molecule destroy` terminate test instance and start from a clean slate.
        ```bash
        $ molecule converge
        $ molecule verify
        $ molecule destroy
        ```
    - If required to log into the Molecule testing instance during development use `molecule login`,
        ```bash
        $ molecule login --host <platform_name>
        ```
    - Ansible modules `fail`, `debug` and `assert` can be use to extract certain information or set breakpoints to troubleshoot errors.
10. After role development is complete run a full Molecule test life cycle.
    ```bash
    $ molecule test
    ```
11. Molecule testing with GitHub Actions (CI)
    - Create the workflow to trigger during *push* and *pull requests* to main branches.
    - Example of YAML configuration file for Github Action workflow CI testing `.github/workflow/ci.yml`,
        ```yaml
        ---
        name: CI
        'on':
          pull_request:
          push:
            branches:
              - master

        jobs:
          Ansible-role-test:
            name: Molecule
            runs-on: ubuntu-latest
            strategy:
              matrix:
                distro:
                  - centos8
                  - ubuntu2004
                  - debian10
            steps:
              - name: Check out the codebase.
                uses: actions/checkout@v2

              - name: Set up Python 3.
                uses: actions/setup-python@v2
                with:
                  python-version: '3.x'

              - name: Install test dependencies.
                run: pip3 install -r requirements.txt

              - name: Run Molecule tests.
                run: |
                  cd myrole
                  molecule test
                env:
                  PY_COLORS: '1'
                  ANSIBLE_FORCE_COLOR: '1'
                  MOLECULE_DISTRO: ${{ matrix.distro }}
        ```

## References
- [ansible-molecule-testing](https://github.com/universalvishwa/ansible-molecule-testing)