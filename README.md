<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/othneildrew/Best-README-Template">
    <img src="resources/flask.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">AWS Ansible Experiment Suite</h3>

  <p align="center">
    An experiment suite for running client-server experiments on AWS EC2 instances.
  </p>
</p>



<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

The AWS Ansible Experiment Suite automates the process of running experiments on AWS. 
On a high level, the experiment suite creates AWS resources (VPC, EC2 instances), installs required packages, and builds the artifact.

Afterwards, the suite sequentially executes jobs of an experimental design (DOE).
The suite supports a multi-factor and multi-level experiment design with repetition.
(i.e., it is possible to vary multiple parameters and repeat each run)
A `YAML`file in `experiments/designs` describes the full experiment design.

Finally, after completing the experiment (all jobs) the suite can cleanup the created AWS resources.

### Built With

This section should list any major frameworks that you built your project using. Leave any add-ons/plugins for the acknowledgements section. Here are a few examples.
* [Ansible](https://www.ansible.com/)
* [YAML](https://yaml.org/)


<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

* Before starting, ensure that you have `pipenv` installed:
    ```sh
    pip install pipenv
    ```

* Moreover, create a `key pair for AWS` [(see instructions)](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-keypair.html)
* and ensure that you can clone remote `repositories with SSH` [(see instructions for GitHub)](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh).



### Installation

1. Create repository from this template
2. Clone newly created repository
    ```sh
    git clone https://github.com/<YOUR REPOSITORY>.git
    ```
3. Install the Python packages (for Ansible)

    ```sh
    pipenv install
    ```

4. Configure `ssh` and `ssh-agent`
 
  * Configure ~/.ssh/config:  (add to file and replace the key for AWS for example with aws_ppl.pem)
      ```
      Host ec2*
      IdentityFile ~/.ssh/{{ exp_base.key_name }} 
      ForwardAgent yes
      ```
  * Add the GitHub private key to ssh-agent:
    ssh-agent ~/.ssh/private_key_rsa
    ssh-add ~/.ssh/private_key_rsa

  * (On a MAC, need add to keychain)

5. Install AWS CLI (version 2) and configure boto
* for boto set the aws credentials in `.boto`
* potentially also check or set credentials in `~/.aws/credentials`

6. Install Ansible collections 

```sh
ansible-galaxy install -r requirements-collections.yml
```

7. Run the repository initialization helper script and configure the experiment suite.
(prompts user input to perform variable substitution in the template [group_vars/all/main.yml.j2](group_vars/all/main.yml.j2)

When unsure set the unique `project id` and the AWS `key name` from the prerequisites and otherwise use the default options.

```sh
pipenv run python scripts/repotemplate.py
```

8. Try running the example experiment design (see [experiments/designs/example.yml](experiments/designs/example.yml))

```sh
ansible-playbook experiment.yml -e "exp=example id=new"
```



<!--


### Setup Environment
* create VPC
* create a set of client EC2 instances and create a set of server EC2 instances
* install packages on EC2 instances

### Experiment Design


### Experiment Run


### Experiment Job

A job is single run of the benchmark with one specific configuration. 
The relation between an experiment run and a job is that a run is repeated multiple times (see `n_repetitions` in experiment design).


The jobs are derived from the experiment design in `experiment/designs`.
The design contains a base configuration (`base_experiment`) that marks which options are factors of the experiment (i.e., what is varied in the experiment).
Moreover, the design contains a list of runs (`factor_levels`) that specifies the level of the factor in the run (i.e., what concrete value to use for a factor in a particular run).

In combination with number of repetitions (see `n_repetitions` in experiment design), the suite derives a list of jobs for the experiment.

`<RUN>_<REP>`

-->



<!-- USAGE EXAMPLES -->
## Usage

TODO: intro 

### Moving beyond Example Experiment

- TODO: todo's in repo for places where things traditionally need to be adjusted
- seen the script to fill variable TODOs 

```sh
pipenv run python scripts/repotemplate.py
```

#### Further Examples

- [pps-lab/bthesis-emanuel-exp-suite](https://github.com/pps-lab/bthesis-emanuel-exp-suite) runs experiments on a single instance and also measures memory usage


### Design of Experiments

The experiment suite runs experiments based on `YAML` files in [experiments/designs](experiments/designs).

An experiment design `YAML` file consists of three parts:
1. The `base_experiment` consists of all the configuration options. All configuration options that vary between runs (i.e., the factors of the experiment) are marked with the placeholder `$FACTOR$`. The remaining configuration options are filled with a constant.

2. The list of `factor_levels` specifies the levels that the factors take in a particular experiment run. For example, in the first run of the experiment, the framework replaces the `$FACTOR$`'s placeholder with the values specified in the first entry in the `factors_levels`list.  

3. The `n_repetitions` variable specifies how many times to repeat each experiment run. (i.e., how many times to execute an experiment run with the same configurations).

```YAML
n_repetitions: 3

base_experiment:
  seed: 1234                  # constant across runs
  payload_size_mb: $FACTOR$   # varies across runs
  opt: $FACTOR$               # varies across runs

factor_levels:
# 3 runs where we vary the factors payload and opt.
# However, for the 1 MB payload, we don't run the opt.
- payload_size_mb: 1
  opt: false
- payload_size_mb: 128
  opt: true
- payload_size_mb: 128
  opt: false

# experiments/designs/xyz.yml
```

#### Table File

For convenience when running a full factorial design, i.e., experiment with every possible combination of all factor levels, we provide a script (see [scripts/expdesign.py](scripts/expdesign.py)) to generate the required config files based on a more concise representation. 

Given a concise experiment configuration with a set of factors and each factor with multiple levels in a "table" form.
The script expands the concise "table" form configuration into the experiment design by performing a cross product of all factor levels.

Simple Example:

TODO [nku] this should be a script rather than a function to call
```python
  build_config_product(exp_name="simple", n_repetitions=3)

  # from scripts/expdesign.py
```

Experiment in "table" form:
```YAML
seed: 1234                  # a constant in the experiment 
payload_size_mb:            # a factor with two levels
  $FACTOR$: [1, 128]
opt:                        # a factor with two levels
  $FACTOR$: [true, false]   

# experiments/table/simple.yml 
```

transforms into the cross product of all factor levels:

Experiment in "design" form:
```YAML
n_repetitions: 3

base_experiment:
  seed: 1234
  payload_size_mb: $FACTOR$
  opt: $FACTOR$

factor_levels:
- payload_size_mb: 1
  opt: true
- payload_size_mb: 1
  opt: false
- payload_size_mb: 128
  opt: true
- payload_size_mb: 128
  opt: false

# experiments/designs/simple.yml
```


See the [scrips-demo.ipynb](scrips-demo.ipynb) for another example.

### Running an Experiment 

We run an experiment by starting the Ansible playbook.
We provide the name of an experiment design from `experiments/designs` (e.g., example) and we use `id=new` to run a new complete experiment.  

```sh
ansible-playbook experiment.yml -e "exp=example id=new"
```

When we start a new experiment, we receive an experiment id (epoch timestamp).

The experiment suite periodically checks whether an experiment run is finished and then starts the next one according to the experiment design. For this we have  

To continue checking we can specify the ID of the experiment.

```sh
ansible-playbook experiment.yml -e "exp=example id=<ID>"
```

For convenience, we can also use `id=last` to continue with the most recent experiment with the provided name:

```sh
ansible-playbook experiment.yml -e "exp=example id=last"
```

### Cleaning up AWS


By default, after an experiment is complete, all resources created on AWS are terminated.
To deactivate this default behaviour, provide the flag: `awsclean=false`

Example:
```sh
ansible-playbook experiment.yml -e "exp=example id=new awsclean=false"
```

Furthermore, we also provide a playbook to terminate all AWS resources:
```sh
ansible-playbook clear.yml
```



### Experimental Results
- folder structure
- helper scripts (see notebook)



<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.



<!-- CONTACT -->
## Contact

Nicolas Küchler - [nicolas-kuechler](https://github.com/nicolas-kuechler)

Project Link: [https://github.com/pps-lab/aws-simple-ansible](https://github.com/pps-lab/aws-simple-ansible)



<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements

