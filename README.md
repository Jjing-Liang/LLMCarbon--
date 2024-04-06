# Title

# Team

# Usage
1. Install Impact Framework and the official plugins.
Install the Impact Framework and the official plugins globally using npm.
```bash
npm install -g @grnsft/if
npm install -g @grnsft/if-plugins
```

2. Run the Example Manifest
Before to build up your owner LLM Carbon manifest, you can run the example manifest to get the breif idea. You can download or copy the manifest yaml file in the [example folder](https://github.com/Jjing-Liang/LLMCarbon--/tree/main/example) as you want.

The `llm-oprational-carbon.yml` is the example manifest for calculate the oprational carbon footprint of LLM, which includes [the data from LLaMA](https://arxiv.org/abs/2302.13971).
You can use the IF Framework command to run the manifest:
```bash
ie --manifest llm-oprational-carbon.yml 
```
If you want to get the output as a yaml file, you can use the following command:
```bash
ie --manifest llm-oprational-carbon.yml --output <result_file_name>
```
You can find the LLM operational carbon in the result:
```yaml
...
...
tree:
  children:
    child:
      pipeline:
        - training-operation-carbon-multiply
      inputs:
        - gpu/num: 992
          training_hour: 82432
          gpu/tdp: 0.4
          pue: 1.1
          carb_inten: 0.385
      outputs:
        - gpu/num: 992
          training_hour: 82432
          gpu/tdp: 0.4
          pue: 1.1
          carb_inten: 0.385
          operation-carbon: 13852268.953600002
```
The operation carbon is 13852268.953600002 kgCO2

The other files in [example folder](https://github.com/Jjing-Liang/LLMCarbon--/tree/main/example) use the [GPT-3 data](https://arxiv.org/abs/2005.14165) and they are:
`llm-carbon-basic.yml`: use the basic formula to calculat oprational carbon and embodied carbon.
`llm-carbon-with-estimated-training-time1.yml` and `llm-carbon-with-estimated-training-time2.yml`: use the formula with estimated LLM training hours to calculat oprational carbon and embodied carbon.

3. Cumpute with customized manifest file
According to your requirements, you can utilize the manifest template files located in the [manifest folder](https://github.com/Jjing-Liang/LLMCarbon--/tree/main/manifest) to compute LLM training carbon emissions.
The files `llm-carbon-basic.yml`, `llm-oprtaional-carbon.yml`, `llm-embodied-carbon.yml`, and `llm-embodied-carbon-using-scim.yml` contain sample input data.

If you lack precise LLM training times but possess detailed LLM model data, such as model parameter numbers and token counts, you can calculate LLM training carbon emissions using estimated training times with the files `llm-carbon-with-estimated-training-time1&2.yml`, `llm-oprtaional-carbon-with-estimated-training-time1&2.yml`, and `llm-embodied-carbon-with-estimated-training-time1&2.yml`.

# Application
For calculating the LLM Carbon Footprint, we employ the basic formula for calculating the carbon footprint: `CO2eq = CO2eq_oper + CO2eq_emb`. The `CO2eq_oper` term represents the carbon footprint of the operation of the LLM, while the `CO2eq_emb` term represents the carbon footprint of the embedding of the LLM.

The fundamental equation for `CO2eq_oper` is `CO2eq_oper = energy_oper * carb_inten`, where `energy_oper` represents the energy utilized during the operation of the LLM, and `carb_inten` denotes the carbon intensity of the energy consumed.

To derive `energy_oper`, the [Watt-hour formula](https://arxiv.org/abs/2111.00364) `energy_oper(Wh) = n * T * TDP * PUE` is employed. Hence, acquiring `energy_oper` depends on the total time for training an LLM, `n` number of GPUs plus the training time `T`, the power consumption of the GPU (Thermal Design Power, TDP), and the Power Usage Effectiveness (`PUE`).

The final equation for operational footprint is:` CO2eq_oper =  n * T * TDP * PUE * carb_inten`

The embodied emissions for training an LLM, denoted as `CO2eq_emb`, are computed as the sum of `CO2eq_emb_i` values for each hardware unit involved in the process.

The embodied emissions for each hardware unit are calculated using the formula: `CO2eq_emb_i = (t_i * CO2eq_chip_i) / lifetime_i`, where `t_i` represents the execution duration of the hardware unit, which equates to the total time required for training an LLM. `CO2eq_chip_i` denotes the CO2 emissions per chip, and `lifetime_i` indicates the expected lifespan of the hardware unit. The chip’s embodied carbon footprint `CO2eq_chip_i` within a specific hardware unit is calculated by `CO2eq_chip_i = area_i * CPA_i`.

This is expressed by the formula: `CO2eq_emb = sum(CO2eq_emb_i)`, where `CO2eq_emb_i` represents the embodied emissions of each respective hardware unit. In essence, the hardware units encompass GPU, CPU, SSD, and DRAM. Thus, the aggregate embodied emissions for training an LLM can be articulated as: `CO2eq_emb = sum(CO2eq_emb_GPU, CO2eq_emb_CPU, CO2eq_emb_SSD, CO2eq_emb_DRAM)`.

Based on the provided formula, the LLM `CO2eq` can be computed using the IF Framework through the IF official plugin.

Here is a basic manifest(you can find it in the repo's [examples folder](https://github.com/Jjing-Liang/LLMCarbon--/tree/main/example)):
```yaml
# llm-carbon-basic.yml
name: llm basic operational emissions manifest
description:
  "
  GTP-3 data resource: https://arxiv.org/abs/2005.14165
  
  CO2eq_oper =  n * T * TDP * PUE * carb_inten
  CO2eq_emb = sum(CO2eq_emb_GPU, CO2eq_emb_CPU, CO2eq_emb_SSD, CO2eq_emb_DRAM)
  CO2eq_emb_i = (t_i * area_i * CPA_i) / lifetime_i
  
  T: training hour(training_hour)
  n: number of gpus(gpu/num)
  TDP: power consumption of the GPU(gpu/tdp)
  PUE: Power Usage Effectiveness(pue)
  carb_inten: carbon intensity of the energy consumed(carb_inten)
  "
tags:
initialize:
  outputs:
    - yaml
  plugins:
    training-operation-carbon-multiply:
      method: Multiply
      path: '@grnsft/if-plugins'
      global-config:
        input-parameters: ['gpu/num', 'training_hour', 'gpu/tdp', 'pue', 'carb_inten']
        output-parameter: 'operation-carbon'
    device-expected-lifespan-hours-per-year-multiply:
      method: Multiply
      path: '@grnsft/if-plugins'
      global-config:
        input-parameters: ['expected-lifespan', 'days-per-year', 'hours-per-day']
        output-parameter: 'expected-lifespan-duration'
    reserved-device-hour-with-device-expected-lifespan-divide:
      method: Divide
      path: '@grnsft/if-plugins'
      global-config:
        numerator: 'training_hour'
        denominator: 'expected-lifespan-duration'
        output: 'expected-lifespan-rate'
    gpu-embodied-carbon-multiply:
      method: Multiply
      path: '@grnsft/if-plugins'
      global-config:
        input-parameters: ['gpu/num', 'expected-lifespan-rate','gpu/cap', 'gpu/area']
        output-parameter: 'gpu-carbon-embodied'
    cpu-embodied-carbon-multiply:
      method: Multiply
      path: '@grnsft/if-plugins'
      global-config:
        input-parameters: ['hardware-unit-num', 'expected-lifespan-rate','cpu/cap', 'cpu/area']
        output-parameter: 'cpu-carbon-embodied'
    ssd-embodied-carbon-multiply:
      method: Multiply
      path: '@grnsft/if-plugins'
      global-config:
        input-parameters: ['hardware-unit-num', 'expected-lifespan-rate', 'ssd/cap', 'ssd/area']
        output-parameter: 'ssd-carbon-embodied'
    dram-embodied-carbon-multiply:
      method: Multiply
      path: '@grnsft/if-plugins'
      global-config:
        input-parameters: ['hardware-unit-num', 'expected-lifespan-rate', 'dram/cap', 'dram/area']
        output-parameter: 'dram-carbon-embodied'
    embodied-carbon-sum:
      method: Sum
      path: '@grnsft/if-plugins'
      global-config:
        input-parameters: [ 'gpu-carbon-embodied', 'cpu-carbon-embodied', 'ssd-carbon-embodied', 'dram-carbon-embodied' ]
        output-parameter: 'carbon-embodied'
    llm-carbon-sum:
      method: Sum
      path: '@grnsft/if-plugins'
      global-config:
        input-parameters: [ 'carbon-embodied',  'operation-carbon']
        output-parameter: 'total-carbon'

tree:
  children:
    child:
      pipeline:
        - training-operation-carbon-multiply
        - device-expected-lifespan-hours-per-year-multiply
        - reserved-device-hour-with-device-expected-lifespan-divide
        - gpu-embodied-carbon-multiply
        - cpu-embodied-carbon-multiply
        - ssd-embodied-carbon-multiply
        - dram-embodied-carbon-multiply
        - embodied-carbon-sum
        - llm-carbon-sum
      defaults:
        thousands-per-unit: 0.001
        days-per-year: 365
        hours-per-day: 24
        seconds-per-hour: 3600
        expected-lifespan: 5
      inputs:
        - gpu/num: 10000
          training_hour: 355.2 # 14.8 days
          gpu/tdp: 0.3 # 300 Watts
          pue: 1.1
          carb_inten: 0.429 # CO2eq/KWh the carbon intensity of training region
          gpu/cap: 1.2    # kgC02/cm2
          gpu/area: 8.15  # cm2
          cpu/cap: 1      # kgC02/cm2
          cpu/area: 1.47  # cm2
          ssd/cap: 0.024  # kgC02/GB
          ssd/area: 32768 # GB 32TB
          dram/cap: 0.4   # kgC02/GB
          dram/area: 256  # GB
          hardware-unit-num:  1250 # gpu_num / 8  assuming one CPU, SSD, DRAM for every 8 GPU/TPU chip or one server stack
```
In the manifest, we use the [GPT-3 data](https://arxiv.org/abs/2005.14165). Methods such as Sum, Multiply, and Divide from the IF official plugin are utilized to construct the formula. 

You can use the IF Framework command to run the manifest:
```bash
ie --manifest llm-carbon-basic.yml 
```
If you want to get the output as a yaml file, you can use the following command:
```bash
ie --manifest llm-carbon-basic.yml --output <result_file_name>
```
The result will be as follows:

```yaml
...
...
tree:
  children:
    child:
      pipeline:
        - training-operation-carbon-multiply
        - device-expected-lifespan-hours-per-year-multiply
        - reserved-device-hour-with-device-expected-lifespan-divide
        - gpu-embodied-carbon-multiply
        - cpu-embodied-carbon-multiply
        - ssd-embodied-carbon-multiply
        - dram-embodied-carbon-multiply
        - embodied-carbon-sum
        - llm-carbon-sum
      defaults:
        thousands-per-unit: 0.001
        days-per-year: 365
        hours-per-day: 24
        seconds-per-hour: 3600
        expected-lifespan: 5
      inputs:
        - gpu/num: 10000
          training_hour: 355.2
          gpu/tdp: 0.3
          pue: 1.1
          carb_inten: 0.429
          gpu/cap: 1.2
          gpu/area: 8.15
          cpu/cap: 1
          cpu/area: 1.47
          ssd/cap: 0.024
          ssd/area: 32768
          dram/cap: 0.4
          dram/area: 256
          hardware-unit-num: 1250
      outputs:
        - gpu/num: 10000
          training_hour: 355.2
          gpu/tdp: 0.3
          pue: 1.1
          carb_inten: 0.429
          gpu/cap: 1.2
          gpu/area: 8.15
          cpu/cap: 1
          cpu/area: 1.47
          ssd/cap: 0.024
          ssd/area: 32768
          dram/cap: 0.4
          dram/area: 256
          hardware-unit-num: 1250
          thousands-per-unit: 0.001
          days-per-year: 365
          hours-per-day: 24
          seconds-per-hour: 3600
          expected-lifespan: 5
          operation-carbon: 502856.64
          expected-lifespan-duration: 43800
          expected-lifespan-rate: 0.008109589041095891
          gpu-carbon-embodied: 793.1178082191782
          cpu-carbon-embodied: 14.9013698630137
          ssd-carbon-embodied: 7972.050410958906
          dram-carbon-embodied: 1038.0273972602743
          carbon-embodied: 9818.09698630137
          total-carbon: 512674.7369863014
```
We can see that the `CO2eq_oper` is 502856.64 kgCO2e, the `CO2eq_emb` is 9818.09698630137 kgCO2e, and the total carbon footprint of the model `CO2eq` is 512674.7369863014 kgCO2e.

Sometimes we want to estimate an existing LLM model's Emissions and find that we don't have the exact values for the training hours. We can use the equation `T = C / ( n * FLOP_peak * eff)` to estimate the training hours. 

Basically, the operational emissions of an LLM model combines the training emissions and the inference emissions. To get the operational emissions, we need to estimate the training hours and the inference hours based on this equation. Where `C` represents the computation required, in total floating point operations, `FLOP_peak` represents the device peak throughput, `eff` represents efficiency of the device. 

For the computation required for training, we can use the formula `C_train ≈ 6PD` with parameter size `P` and the training dataset size `D` (tokens). For the computation required for inference, we can use the formula `C_inference ≈ 2P * D_inference`, where `D_inference` means inference dataset size (tokens).

You can find the manifest file in the repo's [example folder](https://github.com/Jjing-Liang/LLMCarbon--/tree/main/example) as `llm-carbon-with-estimated-training-time1.yml` and `llm-carbon-with-estimated-training-time2.yml`. 










