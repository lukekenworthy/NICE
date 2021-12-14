# NICE: Neural network Integer programming Coefficient Extraction
Official codebase for our [paper](https://arxiv.org/abs/2109.12171) "NICE: Robust Scheduling through Reinforcement Learning-Guided Integer Programming", recently accepted to [AAAI 2022](https://aaai.org/Conferences/AAAI-22/), one of the top AI conferences.

I wrote this code largely in collaboration with [Siddharth Nayak](https://github.com/nsidn98). We had some additional code contribution from [Travis Smith](https://github.com/Travis42). My contributions to this codebase involve all of the code extracting weights from the neural network to be used in the integer program, which is the main contribution of the paper. I also contributed the code that ran the experiments, the experimental scripts, a large share (at least 50%) of pilot scheduling RL environment. Finally, I created all of the linear programming formulators and solvers, though I adapted much of that code from Kenrick, Jessamyn, and Chris, my colleagues at the MIT-Air Force Artificial Intelligence Accelerator. Siddharth created all of the PPO RL code and the wandb integration.

# Abstract:
Integer  programs  provide  a  powerful  abstraction  for  representing a wide range of real-world scheduling problems. Despite their ability to model general scheduling problems, solving large-scale integer programs (IP) remains a computational challenge  in  practice.  The  incorporation  of  more  complex objectives  such  as  robustness  to  disruptions  further  exacerbates the computational challenge. We present NICE (Neural network IP Coefficient Extraction), a novel technique that combines reinforcement learning and integer programming to tackle  the  problem  of  robust  scheduling.  More  specifically, NICE uses reinforcement learning to approximately represent complex objectives in an integer programming formulation. We use NICE to determine assignments of pilots to a flight crew schedule so as to reduce the impact of disruptions. We compare NICE with (1) a baseline integer programming formulation  that  produces  a  feasible  crew  schedule,  and  (2) a robust integer programming formulation that explicitly tries to minimize the impact of disruptions. Our experiments show that NICE produces schedules that are more robust to disruptions than the baseline formulation, with computation times that are lower than those of the robust integer program.

## Disruption experiment structure

Experiemental code for NICE logits scheduling approach. Disruption experiments are run with:

`python3 test_disrupt.py -l feasibility -i 100 -p model_wts/s0_d1_m0.0_b1.0.ckpt -d flights_delay -n 50 3 -r 2 -c 2 > constrained_experiment/delayed_50_buffer_exp.txt`

`-l` specifies the type of baseline linear program you are comparing the logits approach to. "feasibility" in this case refers to the baseline IP.

`-i` specifies the number of iterations it should be run for.

`-p` is the saved neural network to use for the NICE and RL scheduling approach.

`-d` specifies the disruption type. "flights_delay" indicates that flights will be delayed

`-n` specifies the parameters for the disruption type. The first value ("50") is the percentage of flights that get delayed. The second ("3") is the maximum number of days any one flight will be delayed.

`-r` inidicates the "n" value for the weight extraction method

`-c` indicates the density of flights for the given schedule.


Note that we changed the name of the NICE scheduler while developing the code. In the codebase, it is referred to as "CARLO", "LOGITS", and "NICE". They are all synonymous. 

### Bash commands and corresponding experiments:

For the model training (all of the experiments in the `training_scripts` folder), we used the Python package versions found in `training_requirements.txt` with Python 3.7.3. All of the other experiments used the package versions in `testing_requirements.txt` with Python 3.8.2.

- `training_scripts/run_exps.sh`: script to train all of the NICE neural networks.
- `time_experiments`:
  - `run_exp_{X}.sh` runs the experiments where the buffer IP failed to create a single schedule
  - `bufferX.txt` shows the results of those experiments (no output is produced because the experiments never finished running)
- `selection_experiments`:
  - `generate_commands.py` produces the `run_exps.sh` file
  - `parse.py` extracts all of the data out of the files produced by the experiment.
  - `parsed.txt` contains the parsed data
  - `s{A}_d{B}_m{C}_b{D}_n{E}.txt` contains all of the experimental data. The number of disruptions produced is listed at the end. Note that half of these experiments used the move-up reward function, which we use in the technical appendix of the paper. `A` indicates the seed value. `B` indicates the scheduling density of the experiment. `C` and `D` indicate the reward function used (`C = 1.0` and `D = 0.0` indicates the move-up reward function. `C = 0.0` and `D = 1.0` indicates the buffer reward function.) `E` indicates the `n` value used for weight extraction in the NICE approach. 
  - `run_exps.sh` runs all of the experiments where we ran each network/`n` value combo for 20 iterations to pick the best scheduling method. This is the experiment run in the Scheduling Parameter Selection section of the paper.
- `robustness_experiments`:
  - `run_exps.sh` runs the experiments that produced the data listed in the baseline scheduling scenario. It also has the experiments used for the move-up scheduling in the technical appendix of the code. 
  - `delayed_{X}_{Y}_exp[_v{Z}].txt` is the result of the experiment where `X`% of flights are delayed, and we use the NICE/RL scheduler using the network trained with the `Y` (`buffer` or `moveup`) reward function. If `[_v{Z}]` is included, that means, rather than the baseline IP, the NICE/RL schedulers were compared against the `Z` (`Buffer` or `Moveup`) IP scheduler. Note that, when `Y` or `Z` is `moveup` or `Moveup`, that experiment was included in the technical appendix of the paper.
-  `constrained_experiment`:
   -  `run_exp.sh` runs the experiments in the constrained scheduling environment (where the scheduling density is 2).
   -  `delayed_{X}_buffer_exp.txt` is the result of the experiment where `X`% of the flights are delayed. We did not compare against the buffer IP because, as the time experiments show, the buffer cannot produce a single schedule within 90 minutes of running in this environment.

If you find this code useful, please consider citing our paper:
```
@misc{kenworthy2021nice,
      title={NICE: Robust Scheduling through Reinforcement Learning-Guided Integer Programming}, 
      author={Luke Kenworthy and Siddharth Nayak and Christopher Chin and Hamsa Balakrishnan},
      year={2021},
      eprint={2109.12171},
      archivePrefix={arXiv},
      primaryClass={cs.LG}
}
```
