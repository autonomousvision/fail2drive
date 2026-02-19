# Fail2Drive Plugin Branch
This branch prodives the minimal required files to use Fail2Drive in an existing project. If you don't have an existing CARLA project, we provide a standalone installation with multiple agents [here](https://github.com/autonomousvision/fail2drive?tab=readme-ov-file#installation).

## Installation
If your existing project already uses the Bench2Drive or original leaderboard and scenario runner, no additional packages should be required. Using Fail2Drive is only a matter of selecting the correct versions of these folders and running the fail2drive simulator.

> NOTE: Depending on your CARLA installation, the carla package installed in your environment may have to be changed. In most cases, the official CARLA 0.9.15 pip package should be compatible with Fail2Drive. If in doubt, install the Fail2Drive carla whl file found in the simulator folder.

We explain the steps of introducing Fail2Drive in an existing project using the example of [SimLingo](https://github.com/RenzKa/simlingo):

1. Setup the existing project according to the installation notes. For SimLingo, they can be found [here](https://github.com/RenzKa/simlingo?tab=readme-ov-file#setup).

2. Clone this repo into the existing project structure and checkout the `plugin` branch. We recommend leaving the Fail2Drive files in a separate directory, e.g. `SimLingo/Fail2Drive`.

3. Next, download the customized CARLA simulator provided by Fail2Drive.
```
mkdir f2d_carla
curl -L \
  https://huggingface.co/datasets/SimonGer/fail2drive/resolve/main/fail2drive_simulator.tar.gz \
  | tar -xz -C f2d_carla
```

4. The most important step is correctly specifying all environment variables to use the Fail2Drive directories. The following variables have to be overwritten: (Note that the respective project may require additional variables)

```bash
# NOTE: Replace with your paths
export F2D_DIR=/home/simon/simlingo/fail2drive
export CARLA_ROOT=/home/simon/simlingo/fail2drive/f2d_carla

export LEADERBOARD_ROOT=$F2D_DIR/fail2drive_leaderboard
export SCENARIO_RUNNER_ROOT=$F2D_DIR/fail2drive_scenario_runner

# NOTE: The PYTHONPATH used by your project may already include leaderboard and scenario_runner paths. It's best if these are removed.
export PYTHONPATH=$CARLA_ROOT/PythonAPI/carla:$F2D_DIR/fail2drive_leaderboard:$F2D_DIR/fail2drive_scenario_runner:$PYTHONPATH
```

### All done!
Now all that's left is to test if everything works. For this we use the following command:
```bash
# Terminal 1: Start CARLA
cd $CARLA_ROOT
bash CarlaUE4.sh

# Terminal 2: Run Agent
python $LEADERBOARD_ROOT/leaderboard/leaderboard_evaluator.py \
  --routes $F2D_DIR/fail2drive_split/Generalization_PedestriansOnRoad_185.xml \
  --agent [YOUR_AGENT_FILE] \
  --agent-config [YOUR_AGENT_CONFIG]
```

For the SimLingo example, the command looks like this:
```bash
SAVE_PATH=/home/simon/simlingo/viz python $LEADERBOARD_ROOT/leaderboard/leaderboard_evaluator.py \
  --routes $F2D_DIR/fail2drive_split/Generalization_PedestriansOnRoad_185.xml \
  --agent /home/simon/simlingo/team_code/agent_simlingo.py \
  --agent-config "/home/simon/simlingo/checkpoints/epoch=013.ckpt/pytorch_model.pt+f2dtest"
```

### Evaluation
To evaluate your model on the full benchmark, you can use the SLURM evaluation script [slurm_evaluate.py](slurm_evaluate.py). For more information you can reference the [main README](https://github.com/autonomousvision/fail2drive).

### Common errors
`Agent has no attribute 'get_metric_info'`

Some models that only use Bench2Drive, such as SimLingo, will experience an error because `self.get_metric_info()` is only implemented in the Bench2Drive leaderboard agent. We suggest commenting the lines that use this function your agent file. For SimLingo this means commenting lines 792-798 in `agent_simlingo.py`.
