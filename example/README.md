The yaml manifest files in the `example` folder are used to illustrate how to use the manifest template in the `manifest` folder.
For details, please refer to the [content](https://github.com/Jjing-Liang/LLMCarbon--/blob/main/content.md).

# Files
- `llm-oprational-carbon.yml`: using basic input to calculate the LLM oprational carbon footprint(the equation is: `CO2eq_oper =  n * T * TDP * PUE * carb_inten`). The sample data comes from [LLaMA](https://arxiv.org/abs/2302.13971). The `llm-oprational-carbon-result.yaml` is the result.

- `llm-carbon-basic.yml`: using the basic input to calculate the LLM carbon footprint. The sample data comes from [GPT3](https://arxiv.org/abs/2005.14165). The `llm-carbon-basic-result.yaml` is the result.
                                
- `llm-carbon-with-estimated-training-time1.yml`: using the estimated training time(the equation is: `T = C / ( n * FLOP_peak * eff)`) to calculate the carbon footprint. The sample data comes from [GPT3](https://arxiv.org/abs/2005.14165). The `llm-carbon-with-estimated-training-time1-result.yaml` is the result.

- `llm-carbon-with-estimated-training-time2.yml`: using the estimated training time(the equation is: `C / ( n * X)`) to calculate the carbon footprint. The sample data comes from [GPT3](https://arxiv.org/abs/2005.14165). The `llm-carbon-with-estimated-training-time2-result.yaml` is the result.

# Usage
## 1. Install Impact Framework and the official plugins.
Install the Impact Framework and the official plugins globally using npm.
```bash
npm install -g @grnsft/if
npm install -g @grnsft/if-plugins
```

## 2. Run the Example Manifest
You can use the IF Framework command to run the manifest:
```bash
ie --manifest <example_file_name>.yml 
```
If you want to get the output as a yaml file, you can use the following command:
```bash
ie --manifest <example_file_name>.yml --output <result_file_name>
```