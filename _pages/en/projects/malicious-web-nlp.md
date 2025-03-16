---
title: Malicious Web Request Classification using NLP
lang: en
layout: single
author_profile: true
permalink: /en/projects/malicious-web-nlp/
classes: wide
github:
---

# Malicious Web Request Classification using NLP

Classifier for maliscious actions in web application firewall log entries. {% if page.github %} <a href="{{ page.github }}">View on GitHub</a> {% endif %}

## Key Highlights

- Implemented and trained a custom four-layer BERT model on 45k web application firewall log entries, classifying them into one of nine classes of malicious activities.
- Achieved 85.6% accuracy and 0.90 macro F1-score by using a random forest classifier on top of BERT embeddings.

## Objective

The goal of the model is to classify web application firewall log entries into one of nine classes of malicious activities: HOST_Scan, SQL_Injection, Path_Disclosure, Vulnerability_Scan, Leakage_Through_NW, Directory_Indexing, System_Cmd_Execution, Cross_Site_Scripting, and Automatically_Searching_Infor.

**Dataset**: 45k web application firewall log entries (70% train, 20% validation, 10% test)

**Input examples**:

- **HOST_Scan**: "GET /boaform/admin/formLogin?username=user&psd=user HTTP/1.0\r\n\r\n"
- **Cross_Site_Scripting**: "GET /board/board_view?code=%3Cscript%3Eprompt(document.cookie)%3C/script%3E HTTP/1.1\r\nHost: <www.college.school\r\nAccept-Encoding>: identity\r\nCookie: ID=1094200543; designart_site=lbtbqr99b9n4vr0e2en2p5eoh83idq5i\r\nUser-Agent: python-urllib3/1.26.9\r\n\r\n"

## Methodology

The inputs are first pre-processed and tokenized. Then the input tokens are used to train a BERT model. Usning the embeddings from the BERT model, a random-forest classifier is used for the final classification.

### Pre-processing

The preprocessing includes the following steps:

- convert everything to lowercase
- replace \r\n into a single space
- decode url encoding, e.g. convert %40 back to @
- add spaces before and after punctuations and special characters

**Before**: "GET /board/board_view?code=%3Cscript%3Eprompt(document.cookie)%3C/script%3E HTTP/1.1\r\n"

**After**: "get  / board / board _view ? code =  < script > prompt ( document . cookie )  <  / script >  http / 1 . 1 "

### Tokenization

wordpiece? fasttext?

### BERT Model

<figure style="width: 1000px" class="align-center">
  <img
    src="{{ site.url }}{{ site.baseurl }}/assets/images/malicious-web-nlp/model.jpg"
    alt="model architecture">
  <figcaption>Model architecture. FIXME</figcaption>
</figure>

### Random-Forest Classifier

## Results

- 85.6% accuracy
- 0.90 macro F1-score

#### t-SNE visualization of the BERT embeddings

FIXME, make comment here
<figure style="width: 1000px" class="align-center">
  <img
    src="{{ site.url }}{{ site.baseurl }}/assets/images/malicious-web-nlp/embedding-visualization.jpg"
    alt="embedding-visualization">
  <figcaption>Fig n: t-SNE visualization of BERT embedding.</figcaption>
</figure>

## Limitations and Future Work

- The main limitation is that the model does not take advantage of any domain knowledge. For example, a log entry containing a `<script>` tag is likely a cross-site scripting attack.
- By using domain knowledge to engineer rule-based features, these features can be concatenated to the BERT embeddings to improve performance.

<br><br>
<a href="{{ site.url }}{{ site.baseurl }}/en/projects/">← Back to 'Projects'</a>
