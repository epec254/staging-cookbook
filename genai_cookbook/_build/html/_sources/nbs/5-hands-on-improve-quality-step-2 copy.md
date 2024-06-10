### Implement and evaluate changes

```{image} ../images/5-hands-on/workflow_iterate.png
:align: center
```
<br/>

#### Requirements

1. Based on your [root cause analysis](./5-hands-on-improve-quality-step-1.md), you have identified a potential fix to implement and evaluate
      - Potential fixes are outlined in the [fixing retrieval quality](./5-hands-on-improve-quality-step-1-retrieval.md) or [fixing generation quality](./5-hands-on-improve-quality-step-1-generation.md) sections
2. Your POC application (or another baseline chain) is logged to an MLflow Run w/ a Quality Lab evaluation stored in the same Run

#### Overview
<!--
When working to improve the quality of the RAG system, changes can be broadly categorized into three buckets:

1. **![Data pipeline](../images/5-hands-on/data_pipeline.png)** Data pipeline changes
2. **![Chain config](../images/5-hands-on/chain_config.png)** RAG chain configuration changes
3. **![Chain code](../images/5-hands-on/chain_code.png)** RAG chain code changes

Depending on the specific issue you are trying to address, you may need to apply changes to one or both of these components. In some cases, simultaneous changes to both the data pipeline and RAG chain may be necessary to achieve the desired quality improvements.


##### Data pipeline changes

**Data pipeline changes** involve modifying how input data is processed, transformed, or stored before being used by the RAG chain. Examples of data pipeline changes include (and are not limited to):

- Trying a different chunking strategy
- Iterating on the document parsing process
- Changing the embedding model

Implementing a data pipeline change will generally require re-running the entire pipeline to create a new vector index. This process involves reprocessing the input documents, regenerating the vector embeddings, and updating the vector index with new embeddings and metadata.

##### RAG chain & code changes

**RAG chain changes** involve modifying steps or parameters of the RAG chain itself, without necessarily changing the underlying vector database. Examples of RAG chain changes include (and are not limited to):

- Changing the LLM
- Modifying the prompt template
- Adjusting the retrieval component (e.g., number of retrieval chunks, reranking, query expansion)
- Introducing additional processing steps such as a query understanding step

RAG chain updates may involve editing the **RAG chain configuration file** (e.g., changing the LLM parameters or prompt template), *or* modifying the actual **RAG chain code** (e.g., adding new processing steps or retrieval logic).
-->



As a reminder, there are 3 types of potential fixes:
1. **![Data pipeline](../images/5-hands-on/data_pipeline.png)** [Adjust the data pipeline](#1-adjust-the-data-pipeline)
    - *e.g., change the chunk sizes, parsing approach, etc*
    - *Note: To reflect the changed data pipeline, you will need to update the chain's configuration to point to the experiment's vector search*
2. **![Chain config](../images/5-hands-on/chain_config.png)** [Adjust the chain's configuration](#2-adjust-the-chains-configuration)
    - *e.g., change the prompt, retrieval parameters, etc*
3. **![Chain code](../images/5-hands-on/chain_code.png)** [Adjust the chain's code](#3-adjust-the-chains-code)
    - *e.g., add a re-ranker, guardrails, etc*

For all types, you will use the `B_quality_iteration/02_evaluate_experiments` Notebook to evaluate the resulting chain versus your baseline configuration (at first, this is your POC) and pick a "winner".  This notebook will help you pick the winning experiment and deploy it to the Review App or a production-ready, scalable REST API.

At a high level, you will:
1. Implement the fix
2. Evaluate the fix
3. If the fix results in a higher quality chain, deploy the resulting chain to the Review App or as a production-ready REST API.

#### Instructions

1. Follow the linked instructions to implement your change and integrate it into `B_quality_iteration/02_evaluate_experiments`
2. Run the `B_quality_iteration/02_evaluate_experiments` Notebook to evaluate the change

##### 1. **![Data pipeline](../images/5-hands-on/data_pipeline.png)** Adjust the data pipeline

### At a high level, you:
1. Configure and run a modified data pipeline to create a new Vector Index
2. Adjust your chain's configuration to use the the new Vector Index
3. Run evaluation on your updated chain to determine if your change improved quality/cost/latency

### Detailed steps
1. Decide which approach you want to use.
    1. **Run a single experiment at a time:** Allows you to configure and try a single data pipeline at once.  This mode is best if you want to try a single embedding model, test out a single new parser, etc.  We suggest starting here to get familiar with these notebooks.
    2. **Run multiple experiments at once:** Also called a sweep, this approach allows you to, in parallel, execute multiple data pipelines at once.  This mode is best if you want to "sweep" across many different strategies, for example, evaluate 3 PDF parsers or evaluate many different chunk sizes.
2. Open the appropriate README and follow the steps there to create a new data pipeline and vector index.
    1. **Run a single experiment at a time:** [data_pipeline_experiments/single_experiment/README.md](./data_pipeline_experiments/single_experiment/README.md)
    2. **Run multiple experiments at once:** [data_pipeline_experiments/sweeps/README.md](./data_pipeline_experiments/sweeps/README.md)
3. After running the new data pipeline, the adjusted chain configuration will be stored inside an MLflow Run in your app's MLflow experiment.
4. Open the [01_evaluate_experiments](01_evaluate_experiments) Notebook and follow the steps there to run evaluation.


## 2. Adjust the chain's configuration

At a high level, you:
1. Adjust your chain's configuration
2. Run evaluation on your updated chain to determine if your change improved quality/cost/latency

### Detailed steps
1. Open the [01_evaluate_experiments](01_evaluate_experiments) Notebook and follow the steps there to create a modified chain configuration and run evaluation.

## 3. Adjust the chain's code

At a high level, you:
1. Modify your chain's code, optionally parameterizing the changes
2. Update the configuration to match the new parameters
3. Run evaluation on your updated chain to determine if your change improved quality/cost/latency

In this repo, we include several common chain code changes that can improve quality:
- **TODO** *Query rewriting:* Translating a user’s query into 1 or more queries that better represent the original intent in order to make the retrieval step more likely to find relevant documents. 
- **TODO** *Filter extraction:* Extracting specific filters from a user’s query that can be passed to the retrieval step.  This must be done in conjunction with changes to data pipeline to extract this metadata.
- *Re-ranking*: Retrieving a greater number of chunks, and then re-ranking them to identify a smaller number of most relevant chunks

### Detailed steps
1. Create a modified chain code file
2. Open the [01_evaluate_experiments](01_evaluate_experiments) Notebook and follow the steps there to evaluate the updated chain code.





#### Testing a potential fix that could improve quality

Once you have identified a potential fix based on the debugging process outlined above, follow these steps to test your changes:

1. Make the necessary changes to the data pipeline or RAG chain code
   - See the [code examples](#code-examples) below for how and where to make these changes
   - If required, re-run the data pipeline to update the vector index with the new embeddings and metadata

2. Log a new version of your chain to MLflow
   - Ensure that any config files (i.e., for both your data pipeline and RAG chain) are logged to the MLflow run
     - <SCREENSHOT OF LOGGED CHAIN>

3. Run evaluation on this new chain

4. Review evaluation results
   - Analyze the evaluation metrics to determine if there has been an improvement the RAG chain's performance
     - <SCREENSHOT OF EVALS>
   - Compare the traces and LLM judge results for individual queries before and after the changes to gain insights into the impact of your changes

5. Iterate on the fixes
   - If the evaluation results do not show the desired improvement, iterate on your changes based on the insights gained from analysis.
   - Repeat steps 1-4 until you are satisfied with the improvement in the RAG chain's output quality

6. Deploy the updated RAG chain for user feedback
   - Once evaluation results indicate improvement, register the chain to Unity Catalog and deploy the updated RAG chain via the Review App.
   - Gather feedback from stakeholders and end-users through one or both of the following:
     - Have stakeholders interact with the app directly in the RAG Studio UI and provide feedback on response quality
       - <SCREENSHOT OF REVIEW APP>
     - Generate responses using the updated chain for the set of evaluation queries and seek feedback on those specific responses

7. Monitor and analyze user feedback
   - Review these results using a [dashboard](https://docs.databricks.com/en/dashboards/index.html#dashboards).
   - Monitor metrics such as the percentage of positive and negative feedback, as well as any specific comments or issues raised by users.

<!--
### Code Examples
| | Component | Change(s) |
|---|---|---|
| **Data pipeline changes**<br><br>1. Re-run data pipeline to create new vector index<br>2. Log new version of RAG chain using the updated index<br>3. Run evals on new chain | [Parser](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/02_parse_docs.py) | - Change parsing strategy<br>  - [Add new parsing strategy to notebook](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/parser_library.py)<br>  - [Update data pipeline config](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/00_config.py#L29) |
| | [Chunking](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/03_chunk_docs.py) | - Chunking strategy<br>  - [Add or update existing chunking strategy](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/chunker_library.py)<br>  - [Update data pipeline config](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/00_config.py#L30-L35)<br>- Change chunk sizes of existing chunking strategy<br>  - [Update data pipeline config](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/00_config.py#L30-L35)<br>- Add metadata to chunks<br>- Semantic chunking |
| | [Embedding<br>model](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/04_vector_index.py#L41) | - Change embedding model<br>  - [Update data pipeline config](https://github.com/databricks-field-eng/field-ai-examples/blob/main/dev/data_processing/notebook_version/data_prep/00_config.py#L18-L24) |
| **RAG chain config changes**<br><br>1. If no changes to data pipeline, do *not* re-run data pipeline<br>2. Log new version of RAG chain using the updated index<br>3. Run evals on new chain | [LLM](#llm) | - Change LLM or its parameters<br>  - [Update RAG chain config](https://github.com/epec254/rag_code/blob/main/RAG%20Cookbook/B_pdf_rag_with_multi_turn_chat/2_rag_chain_config.yaml#L1-L4) |
| | [Prompt<br>Template](/nbs/3-deep-dive.md#prompt-augmentation) | - Iterate on prompt template<br>  - [Update RAG chain config](https://github.com/epec254/rag_code/blob/main/RAG%20Cookbook/B_pdf_rag_with_multi_turn_chat/2_rag_chain_config.yaml#L5-L14) |
| | [Hybrid search](/nbs/3-deep-dive.md#retrieval) | - Try hybrid search instead of semantic search<br>  - Update RAG chain code |
| **RAG chain code changes**<br><br>1. If no changes to data pipeline, *do not* re-run data pipeline<br>2. Log new version of RAG chain using the updated index<br>3. Run evals on new chain | [Reranker](/nbs/3-deep-dive.md#retrieval) | - Add reranker step to RAG chain<br>  - [Update RAG chain code](https://github.com/epec254/rag_code/pull/19) |
| | [Query<br>Expansion](/nbs/3-deep-dive.md#query-understanding) | - Add query expansion step<br>  - Update RAG chain code<br>  - NOTE: Implement this [example prompt](https://docs.llamaindex.ai/en/stable/examples/query_transformations/query_transform_cookbook/#query-rewriting-custom) into the multi turn |
| | [Guardrails](/nbs/3-deep-dive.md#post-processing-guardrails) | - Add post-processing guardrails step to RAG chain<br>  - Create a version of this [chain](https://github.com/epec254/rag_code/tree/main/RAG%20Cookbook/B_pdf_rag_with_multi_turn_chat) that includes a sample guardrail prompt using the current advanced DBdemo as an example |
-->
