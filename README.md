# This is the data and code repository for manuscripts submitted to EMSE journal.

## Datasets
We first give the datasets under the datasets folder shown above, these datasets were processed well by previous researchers.

## Code Pre-trained Models
The code pre-trained models used in this paper are all come from HuggingFace, we give the links to download them:

CodeBERT-base: https://huggingface.co/microsoft/codebert-base

CodeT5-base: https://huggingface.co/Salesforce/codet5-base

PLBART-csnet: https://huggingface.co/uclanlp/plbart-csnet

CodeGPT: https://huggingface.co/microsoft/CodeGPT-small-py-adaptedGPT2

CodeGen-base: https://huggingface.co/Salesforce/codegen-350M-mono

We do not give the models shown above since it is too large and easy to obtain.


## Different Embedding Ways
The embedding method can be found in detail in the paper, and this step can be easily implemented. 

We provide code blocks to obtain embeddings by special tokens and average-pooling all codes tokens. 

These code chunks can be put into the pipeline where the pre-trained model is applied to downstream tasks (e.g., the pipeline provided by CodeXGLUE) to obtain code embeddings, provided that the model parameters are first frozen.

We assume that the hidden_states of the last layer of the model have been obtained, and the shape may be [batch_size, input_length, dimention].

### 1-encoder-only
Then we can use the next code blocks to get embedding of the first token of encoder-only pre-trained model, for example special token [CLS] of CodeBERT, The premise is that the padding method is the right padding.

    first_special_token_embedding = hidden_states[:,0,:]  # The shape may be [batch_size, dimention]

### 2-encoder-only
We can use the next code blocks to get embedding of all code tokens of encoder-only pre-trained model, The premise is that the padding method is the right padding.

    hidden_states = hidden_states[:, 1:, :] # Remove the vector representation corresponding to the first_special_token
    attention_mask = attention_mask[:, 1:] # Remove the attention corresponding to the first_special_token
    for row in attention_mask:
        last_true_index = (row == True).nonzero(as_tuple=True)[0][-1]
        row[last_true_index] = False # Set attention to the last special token to False
    input_mask_expanded = attention_mask.unsqueeze(-1).expand(hidden_states.size()).float()
    all_code_token_embedding = torch.sum(hidden_states * input_mask_expanded, 1) / torch.clamp(input_mask_expanded.sum(1), min=1e-9) # The shape may be [batch_size, dimention]
  

### 1-decoder-only
We can use the next code blocks to get embedding of the last token of decoder-only pre-trained model, The premise is that the padding method is the left padding.

    first_special_token_embedding = hidden_states[:,-1,:]  # The shape may be [batch_size, dimention]

### 2-decoder-only
We can use the next code blocks to get embedding of all code tokens of decoder-only pre-trained model, The premise is that the padding method is the left padding.

    hidden_states = hidden_states[:,:-1,:] # Remove the vector representation corresponding to the last special token
    attention_mask = attention_mask[:,:-1] # Remove the attention corresponding to the last special token
    input_mask_expanded = attention_mask.unsqueeze(-1).expand(hidden_states.size()).float()
    all_code_token_embedding = torch.sum(hidden_states * input_mask_expanded, 1) / torch.clamp(input_mask_expanded.sum(1), min=1e-9) # The shape may be [batch_size, dimention]


### 1-encoder-decoder
We can use the next code blocks to get embedding of the first token of encoder-decoder pre-trained model, The premise is that the padding method is the right padding.

    first_special_token_embedding = hidden_states[:,0,:]  # The shape may be [batch_size, dimention]

### 2-encoder-decoder
We can use the next code blocks to get embedding of the last token of encoder-decoder pre-trained model, The premise is that the padding method is the right padding.

    last_special_token_embedding = hidden_states[input_ids.eq(model.config.eos_token_id), :].view(hidden_states.size(0), -1, hidden_states.size(-1))[:, -1, :] # Select the location containing the special token from the hidden state based on the special token id
        
### 3-encoder-decoder
We can use the next code blocks to get embedding of all code tokens of encoder-decoder pre-trained model, The premise is that the padding method is the right padding.

    hidden_states = hidden_states[:, 1:, :] # Remove the vector representation corresponding to the first_special_token
    attention_mask = attention_mask[:, 1:] # Remove the attention corresponding to the first_special_token
    for row in attention_mask:
        last_true_index = (row == True).nonzero(as_tuple=True)[0][-1]
        row[last_true_index] = False # Set attention to the last special token to False
    input_mask_expanded = attention_mask.unsqueeze(-1).expand(hidden_states.size()).float()
    all_code_token_embedding = torch.sum(hidden_states * input_mask_expanded, 1) / torch.clamp(input_mask_expanded.sum(1), min=1e-9) # The shape may be [batch_size, dimention]

        

## Acknowledgment
Thank the following works for providing the original repository to facilitate the collection of the data sets required for this paper. At the same time, we gave the address of the raw datasets depository.

Datasets Devign and BigCloneBench for code vulnerability detection and code clone detection
Devign: https://github.com/microsoft/CodeXGLUE/tree/main/Code-Code/Defect-detection
and the raw dataset comes from the paper <b>Devign: Effective Vulnerability Identification by Learning Comprehensive Program Semantics via Graph Neural Networks.</b>

Datasets CWE119 and CWE399: https://github.com/CGCL-codes/VulDeePecker
and these two datasets come from the paper <b>Vuldeepecker: A deep learning-based system for vulnerability detection.</b>

BigCloneBench: https://github.com/microsoft/CodeXGLUE/tree/main/Code-Code/Clone-detection-BigCloneBench
and the raw dataset comes from the paper <b>Detecting Code Clones with Graph Neural Network and Flow-Augmented Abstract Syntax Tree.</b>

Datasets openstack, qt, platform, gerrit, and go for just-in-time defect prediction: https://github.com/ZZR0/ISSTA21-JIT-DP
and the raw dataset comes from the paper <b>Deep just-in-time defect prediction: how far are we?</b>

