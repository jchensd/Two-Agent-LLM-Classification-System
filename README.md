# Two-Agent LLM Classification System

This repository presents a small, complete prototype of an **agentic text classification system** for social science research that requires tasks like social media stance detection.

This is an **ongoing project** aimed at overcoming the limitations of hand-crafted, ad hoc prompt engineering and advancing toward robust, scalable LLM agents that can autonomously manage the entire classification workflow with adaptive decision-making.

## **Motivation**: 

Contemporary research on social phenomena increasingly relies on large-scale text data, such as news articles and social media posts, that traditionally requires extensive human annotation. While recent research has made significant progress in automatic prompt optimization through large language models and reinforcement learning, much of the existing tools focus on optimizing prompts in narrow, static settings. This project is exploring the potential of building a dynamic, self-directed LLM-powered multi-agent system that can actively make decisions and carry out steps throughout the entire classification workflow. This includes not only designing and refining classification prompts, but also planning validation steps, interpreting performance metrics, and adapting policies based on results iteratively.

## **Implementation**: 
The current prototype pipeline is available as a Colab-friendly notebook: [Two_Agent_LLM_Classification_System.ipynb](https://github.com/jchensd/Two-Agent-LLM-Classification-System/blob/63270626064fd822d7b4d53bfcc213ef89cefbb1/Two_Agent_LLM_Classification_System.ipynb)

## **This pipeline**:

1. Preprocesses the labeled training data
   - Resets the index and adds a row_id
   - Cleans the label column (converts to string, strips whitespace)
   - Collects the list of candidate labels and how many times each label appears

2. Asks the controller agent to decide how to inspect the dataset
   - The agent can request:
     - dataset summary
     - first N rows
     - per-label samples
     - a short snippet of Python code it generates to be run on the dataset in a sandbox environment, using only a limited set of safe operations

3. Executes the requested tool and returns the dataset description, label distribution, and the chosen data inspection result to the controller agent

4. Asks the controller agent to design an initial classification prompt for the worker agent and a validation plan

5. Based on the controller agent's decision, builds a label-stratified validation batch by randomly sampling from the dataset and shuffling the sampled rows

6. Directs the worker agent to classify the validation batch
   - For each example, the worker receives:
     - the controller's system prompt
     - the list of candidate labels
     - the input text 
   - It returns a single JSON object with a label field, which is stored in the pred_label column.

7. Evaluates classification performance and extracts misclassifications
   - Computes accuracy, macro-F1, and per-label precision/recall/F1 using scikit-learn
   - Collects a small set of misclassified examples for the controller agent to review
8. Runs an iterative controller-worker loop to refine the prompt
   - At each iteration, the controller agent
     - Sees the current system prompt, classification performance metrics, misclassified examples, and per-label usage statistics
     - Decides whether to accept the current prompt, run additional validation with the same prompt, or revise the prompt and validate again 
     - When requesting more validation, it specifies whether to sample only from unseen rows or from the full dataset, the validation batch sizes, and worker agent temperature (optional)
   - The pipeline carries out these instructions, re-runs the worker, and updates the metrics

9. Evaluates the final prompt on the separate test set
   - If the controller agent accepts the current prompt, this version is fixed as the final classification prompt; otherwise, the last prompt after the maximum number of rounds is used.
   - The pipeline reports test metrics, shows misclassified test examples, and returns the final prompt, and summary performance metrics for further analysis.
  
## **Present Direction**:

A key challenge, based on the experiment, is that the controller agent tends to produce increasingly long and overly detailed classification prompts as the refinement loop continues. This reduces clarity and efficiency for the worker agent and does not improve classification performance. To address this, I am exploring reinforcement learning as a potential approach to optimize the controller agent's policy by encouraging more concise, effective prompts based on the worker agent's behavior.
