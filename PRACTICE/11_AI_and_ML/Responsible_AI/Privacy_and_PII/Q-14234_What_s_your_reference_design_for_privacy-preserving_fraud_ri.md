---
type: "practice"
difficulty: "Expert"
---

# What's your reference design for privacy-preserving fraud/risk models shared across a consortium of banks that legally cannot pool raw data?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Conflicting Privacy Controls](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Conflicting_Privacy_Controls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with designing an architecture to share and leverage AI models for fraud detection or risk assessment while ensuring strict privacy-preserving practices among a consortium of banks. Given the legal constraints, pooling raw data is not an option. Instead, we need to focus on techniques that allow for secure data sharing without revealing sensitive information.

The key challenge here involves both technical implementation and regulatory compliance. The design must ensure data anonymity, confidentiality, and integrity while allowing for effective model training and inference. Let's break down the solution step by step:

Data Aggregation and Encryption: Since raw data pooling is prohibited, each bank will encrypt their individual data using homomorphic encryption or secure multi-party computation (MPC) techniques.
Secure Aggregation Mechanisms: A trusted third party or a decentralized network will be responsible for securely aggregating these encrypted datasets without decrypting them, ensuring that the privacy of each participant's data is maintained.
Model Training and Inference: The aggregated dataset can then be used to train models on the aggregated plaintext or encrypted data (if feasible with current technology). For inference, individual banks will perform their own local predictions using locally held data.

Now let’s delve into the detailed solution:

TL;DR

The reference design involves encrypting raw data at each bank, securely aggregating these encrypted datasets without decryption, and performing model training/inference on aggregated plaintext or encrypted data. This ensures privacy while allowing for effective fraud/risk detection.

#### Triage
What the Question is Really Asking
Objective: Design a system that allows multiple banks to share data for AI models in a way that complies with legal constraints.
Key Probes:
Techniques for secure data aggregation and encryption.
Methods for training and inference on encrypted or aggregated data.
Compliance with relevant privacy laws and regulations.

#### Assumptions
Banks are part of a consortium and share the goal of detecting fraud/risk.
Raw data cannot be pooled due to legal restrictions.
There is an existing trusted third party (TTP) or a decentralized network for coordination.

#### First-Principles Solution
1. Data Aggregation and Encryption

Each bank will encrypt their raw transactional data using advanced cryptographic techniques such as homomorphic encryption (HE) or secure multi-party computation (MPC).

#### Example
Homomorphic Encryption Homomorphic encryption allows computations to be performed on encrypted data without decrypting it, enabling secure aggregation of models. For instance, if we use HE, the bank can encrypt their transaction data and send the ciphertexts to a TTP. The TTP can then perform computations on these ciphertexts and return the result in an encrypted form.

PYTHON
# Simplified Example using HElib (a library for homomorphic encryption)
from helib import *

def encrypt_data(data):
    # Setup parameters
    params = EncryptionParameters()
    scheme = Scheme.BFV
    params.set.scheme(scheme)

    context = SEALContext(params)
    keygen = KeyGenerator(context)
    public_key = keygen.public_key()
    secret_key = keygen.secret_key()

    encoder = Encoder(context)

    # Encrypt data
    plaintext = Plaintext(encoder.encode(data))
    ciphertext = Ciphertext()
    encryptor = Encryptor(context, secret_key)
    encryptor.encrypt(plaintext, ciphertext)

    return ciphertext

# Bank A and Bank B encrypt their respective datasets
bank_a_data_ciphertext = encrypt_data(bank_a_raw_data)
bank_b_data_ciphertext = encrypt_data(bank_b_raw_data)
2. Secure Aggregation Mechanisms

A TTP or a decentralized network will aggregate these encrypted datasets without decrypting them.

#### Example
Secure Aggregation with MPC In this scenario, each bank can perform computations on their own data and share the results in an aggregated form. The TTP can then combine these partial results to get a final model without ever seeing the plaintext data.

PYTHON
# Simplified Example using PySyft (a library for secure multi-party computation)
from syft.frameworks.torch import nn as snn

def aggregate_models(bank_a_model, bank_b_model):
    # Assume each bank has its own local model trained on encrypted data
    # TTP performs aggregation by summing the partial models
    aggregated_model = (bank_a_model + bank_b_model) / 2

    return aggregated_model
3. Model Training and Inference

The aggregated dataset or encrypted model can be used to train a combined fraud/risk detection model.

#### Example
Training on Aggregated Data If the data is aggregated in plaintext, standard training techniques can be applied. If the data remains encrypted, homomorphic encryption might still allow for some form of training.

PYTHON
# Simplified Example using PyTorch with HElib integration
model = snn.Linear(in_features=100, out_features=50)
criterion = snn.MSELoss()
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)

for epoch in range(num_epochs):
    # Perform forward pass on aggregated data (if available)
    output = model(aggregated_data_ciphertext)

    # Compute loss and backpropagate
    loss = criterion(output, target)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the design involves encrypting data at each bank using HE or MPC, securely aggregating encrypted datasets without decryption, and performing model training/inference on aggregated plaintext or encrypted data. Key challenges include ensuring compliance with legal regulations and leveraging advanced cryptographic techniques that may limit computational efficiency.

#### Common mistakes
to avoid:

Not considering regulatory requirements thoroughly.
Ignoring the performance overhead of HE/MPC.
Failing to implement robust security measures during aggregation and inference.

#### Follow-ups
could include more detailed discussions on specific encryption algorithms, implementation challenges, and potential trade-offs between privacy and model accuracy.

