The yaml manifest files in the `manifest` folder are tenplates for calculating the carbon footprint of LLM.
For details, please refer to the [content](https://github.com/Jjing-Liang/LLMCarbon--/blob/main/content.md).


# Usage
## 1. Install Impact Framework and the official plugins.
Install the Impact Framework and the official plugins globally using npm.
```bash
npm install -g @grnsft/if
npm install -g @grnsft/if-plugins
```

## 2. Input your manifest file data
For basic input data, you can use the following manifest files:
- `llm-opertaional-carbon.yml`: caluculate the operational carbon of a LLM
- `llm-emobided-carbon-using-scim.yml`: calculate the embodied carbon of a LLM using IF SCIM Pulugin
- `llm-embodied-carbon.yml`: calculate the embodied carbon of a LLM
- `llm-carbon-basic.yml`: calculate the carbon of a LLM, including the operational carbon and the embodied carbon                                

If you lack precise LLM training times but possess detailed LLM model data, such as model parameter numbers and token counts, you can calculate LLM training carbon emissions using estimated training times with the files:
- `llm-opertaional-carbon-with-estimated-training-time1.yml` and `llm-opertaional-carbon-with-estimated-training-time2.yml`: caluculate the operational carbon of a LLM with estimated training time.
- `llm-carbon-with-estimated-training-time1.yml` and `llm-carbon-with-estimated-training-time2.yml`: caluculate the carbon of a LLM with estimated training time.
- `llm-embodied-carbon-with-estimated-training-time1.yml` and `llm-embodied-carbon-with-estimated-training-time2.yml`: caluculate the embodied carbon of a LLM with estimated training time.

## 3. Run the Example Manifest
You can use the IF Framework command to run the manifest:
```bash
ie --manifest <example_file_name>.yml 
```
If you want to get the output as a yaml file, you can use the following command:
```bash
ie --manifest <example_file_name>.yml --output <result_file_name>
```