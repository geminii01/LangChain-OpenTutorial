<style>
.custom {
    background-color: #008d8d;
    color: white;
    padding: 0.25em 0.5em 0.25em 0.5em;
    white-space: pre-wrap;       /* css-3 */
    white-space: -moz-pre-wrap;  /* Mozilla, since 1999 */
    white-space: -pre-wrap;      /* Opera 4-6 */
    white-space: -o-pre-wrap;    /* Opera 7 */
    word-wrap: break-word;
}

pre {
    background-color: #027c7c;
    padding-left: 0.5em;
}

</style>

# PDF Loader

- Author: [Yejin Park](https://github.com/ppakyeah)
- Design: []()
- Peer Review : [Yun Eun](https://github.com/yuneun92), [MinJi Kang](https://www.linkedin.com/in/minji-kang-995b32230/)
- This is a part of [LangChain Open Tutorial](https://github.com/LangChain-OpenTutorial/LangChain-OpenTutorial)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/LangChain-OpenTutorial/LangChain-OpenTutorial/blob/main/06-DocumentLoader/02-PDFLoader.ipynb) [![Open in GitHub](https://img.shields.io/badge/Open%20in%20GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/LangChain-OpenTutorial/LangChain-OpenTutorial/blob/main/06-DocumentLoader/02-PDFLoader.ipynb)

## Overview
This tutorial covers various PDF processing methods using LangChain and popular PDF libraries.

PDF processing is essential for extracting and analyzing text data from PDF documents.

In this tutorial, we will explore different PDF loaders and their capabilities while working with LangChain's document processing framework.

### Table of Contents

- [Overview](#overview)
- [Environment Setup](#environment-setup)
- [How to load PDFs](#how-to-load-pdfs)
- [PyPDF](#pypdf)
- [PyMuPDF](#pymupdf)
- [Unstructured](#unstructured)
- [PyPDFium2](#pypdfium2)
- [PDFMiner](#pdfminer)
- [PDFPlumber](#pdfplumber)

### References

- [LangChain: How to load PDFs](https://python.langchain.com/docs/how_to/document_loader_pdf/)
----

## Environment Setup

Set up the environment. You may refer to [Environment Setup](https://wikidocs.net/257836) for more details.

**[Note]**
- `langchain-opentutorial` is a package that provides a set of easy-to-use environment setup, useful functions and utilities for tutorials.
- You can checkout the [`langchain-opentutorial`](https://github.com/LangChain-OpenTutorial/langchain-opentutorial-pypi) for more details.

```python
%%capture --no-stderr
%pip install langchain-opentutorial
```

```python
# Install required packages
from langchain_opentutorial import package

package.install(
    [
        "langchain_community",
        "langchain_text_splitters",
        "pypdf",
        "rapidocr-onnxruntime",
        "pymupdf",
        "unstructured[pdf]"
    ],
    verbose=False,
    upgrade=False,
)
```

<pre class="custom">
    [notice] A new release of pip available: 22.3.1 -> 24.3.1
    [notice] To update, run: pip install --upgrade pip
</pre>

```python
# Set environment variables
from langchain_opentutorial import set_env

set_env(
    {
        "OPENAI_API_KEY": "",
        "LANGCHAIN_API_KEY": "",
        "LANGCHAIN_TRACING_V2": "true",
        "LANGCHAIN_ENDPOINT": "https://api.smith.langchain.com",
        "LANGCHAIN_PROJECT": "PDFLoader",
    }
)
```

<pre class="custom">Environment variables have been set successfully.
</pre>

## How to load PDFs

[Portable Document Format (PDF)](https://en.wikipedia.org/wiki/PDF), a file format standardized by ISO 32000, was developed by Adobe in 1992 for presenting documents, which include text formatting and images in a way that is independent of application software, hardware, and operating systems.

This guide covers how to load a PDF document into the LangChain [Document](https://python.langchain.com/api_reference/core/documents/langchain_core.documents.base.Document.html#langchain_core.documents.base.Document) format. This format will be used downstream.

LangChain integrates with a variety of PDF parsers. Some are simple and relatively low-level, while others support OCR and image processing or perform advanced document layout analysis.

The right choice depends on your application.


We will demonstrate these approaches on a [sample file](https://github.com/langchain-ai/langchain/blob/master/libs/community/tests/integration_tests/examples/layout-parser-paper.pdf).
Download the sample file and copy it to your data folder.

```python
FILE_PATH = "./data/layout-parser-paper.pdf"
```

```python
def show_metadata(docs):
    if docs:
        print("[metadata]")
        print(list(docs[0].metadata.keys()))
        print("\n[examples]")
        max_key_length = max(len(k) for k in docs[0].metadata.keys())
        for k, v in docs[0].metadata.items():
            print(f"{k:<{max_key_length}} : {v}")
```

## PyPDF


[PyPDF](https://github.com/py-pdf/pypdf) is one of the most widely used Python libraries for PDF processing.

Here we use PyPDF to load the PDF as an list of Document objects

LangChain's [`PyPDFLoader`](
https://python.langchain.com/api_reference/community/document_loaders/langchain_community.document_loaders.pdf.PyPDFLoader.html) integrates with PyPDF to parse PDF documents into LangChain Document objects.


```python
from langchain_community.document_loaders import PyPDFLoader

# Initialize the PDF loader
loader = PyPDFLoader(FILE_PATH)

# Load data into Document objects
docs = loader.load()

# Print the contents of the document
print(docs[10].page_content[:300])
```

<pre class="custom">LayoutParser: A Uniﬁed Toolkit for DL-Based DIA 11
    focuses on precision, eﬃciency, and robustness. The target documents may have
    complicated structures, and may require training multiple layout detection models
    to achieve the optimal accuracy. Light-weight pipelines are built for relatively
    simple d
</pre>

```python
# output metadata
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source', 'page']
    
    [examples]
    source : ./data/layout-parser-paper.pdf
    page   : 0
</pre>

The `load_and_split()` method allows customizing how documents are chunked by passing a text splitter object, making it more flexible for different use cases.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

# Load Documents and split into chunks. Chunks are returned as Documents.
text_splitter = RecursiveCharacterTextSplitter(chunk_size=200, chunk_overlap=200)
docs = loader.load_and_split(text_splitter=text_splitter)
print(docs[0].page_content)
```

<pre class="custom">LayoutParser: A Uniﬁed Toolkit for Deep
    Learning Based Document Image Analysis
    Zejiang Shen1 (  ), Ruochen Zhang2, Melissa Dell3, Benjamin Charles Germain
    Lee4, Jacob Carlson3, and Weining Li5
</pre>

### PyPDF(OCR)

Some PDFs contain text images within scanned documents or pictures. You can also use the `rapidocr-onnxruntime` package to extract text from images.

```python
# Initialize PDF loader, enable image extraction option
loader = PyPDFLoader(FILE_PATH, extract_images=True)

# load PDF page
docs = loader.load()

# access page content
print(docs[4].page_content[:300])
```

<pre class="custom">LayoutParser: A Uniﬁed Toolkit for DL-Based DIA 5
    Table 1: Current layout detection models in the LayoutParser model zoo
    Dataset Base Model1 Large ModelNotes
    PubLayNet [38] F / M M Layouts of modern scientiﬁc documents
    PRImA [3] M - Layouts of scanned modern magazines and scientiﬁc reports
    Newspaper
</pre>

```python
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source', 'page']
    
    [examples]
    source : ./data/layout-parser-paper.pdf
    page   : 0
</pre>

### PyPDF Directory

Import all PDF documents from directory.

```python
from langchain_community.document_loaders import PyPDFDirectoryLoader

# directory path
loader = PyPDFDirectoryLoader("./data/")

# load documents
docs = loader.load()

# print the number of documents
docs_len = len(docs)
print(docs_len)

# get document from a directory
document = docs[docs_len - 1]
```

<pre class="custom">16
</pre>

```python
# print the contents of the document
print(document.page_content[:300])
```

<pre class="custom">16 Z. Shen et al.
    [23] Paszke, A., Gross, S., Chintala, S., Chanan, G., Yang, E., DeVito, Z., Lin, Z.,
    Desmaison, A., Antiga, L., Lerer, A.: Automatic diﬀerentiation in pytorch (2017)
    [24] Paszke, A., Gross, S., Massa, F., Lerer, A., Bradbury, J., Chanan, G., Killeen,
    T., Lin, Z., Gimelshein, N., An
</pre>

```python
print(document.metadata)
```

<pre class="custom">{'source': 'data/layout-parser-paper.pdf', 'page': 15}
</pre>

## PyMuPDF

[PyMuPDF](https://github.com/pymupdf/PyMuPDF) is speed optimized and includes detailed metadata about the PDF and its pages. It returns one document per page.

LangChain's [`PyMuPDFLoader`](
https://python.langchain.com/api_reference/community/document_loaders/langchain_community.document_loaders.pdf.PyMuPDFLoader.html) integrates with PyMuPDF to parse PDF documents into LangChain Document objects.

```python
from langchain_community.document_loaders import PyMuPDFLoader

# create an instance of the PyMuPDF loader
loader = PyMuPDFLoader(FILE_PATH)

# load the document
docs = loader.load()

# print the contents of the document
print(docs[10].page_content[:300])
```

<pre class="custom">LayoutParser: A Uniﬁed Toolkit for DL-Based DIA
    11
    focuses on precision, eﬃciency, and robustness. The target documents may have
    complicated structures, and may require training multiple layout detection models
    to achieve the optimal accuracy. Light-weight pipelines are built for relatively
    simple d
</pre>

```python
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source', 'file_path', 'page', 'total_pages', 'format', 'title', 'author', 'subject', 'keywords', 'creator', 'producer', 'creationDate', 'modDate', 'trapped']
    
    [examples]
    source       : ./data/layout-parser-paper.pdf
    file_path    : ./data/layout-parser-paper.pdf
    page         : 0
    total_pages  : 16
    format       : PDF 1.5
    title        : 
    author       : 
    subject      : 
    keywords     : 
    creator      : LaTeX with hyperref
    producer     : pdfTeX-1.40.21
    creationDate : D:20210622012710Z
    modDate      : D:20210622012710Z
    trapped      : 
</pre>

## Unstructured

[Unstructured](https://docs.unstructured.io/welcome) is a powerful library designed to handle various unstructured and semi-structured document formats. It excels at automatically identifying and categorizing different components within documents.
Currently supports loading text files, PowerPoints, HTML, PDFs, images, and more.

LangChain's [`UnstructuredPDFLoader`](
https://python.langchain.com/api_reference/unstructured/document_loaders/langchain_unstructured.document_loaders.UnstructuredLoader.html) integrates with Unstructured to parse PDF documents into LangChain Document objects.


```python
from langchain_community.document_loaders import UnstructuredPDFLoader

# create an instance of UnstructuredPDFLoader
loader = UnstructuredPDFLoader(FILE_PATH)

# load the data
docs = loader.load()

# print the contents of the document
print(docs[0].page_content[:300])
```

<pre class="custom">Matplotlib is building the font cache; this may take a moment.
</pre>

    1 2 0 2
    
    n u J
    
    1 2
    
    ]
    
    V C . s c [
    
    2 v 8 4 3 5 1 . 3 0 1 2 : v i X r a
    
    LayoutParser: A Uniﬁed Toolkit for Deep Learning Based Document Image Analysis
    
    Zejiang Shen1 ((cid:0)), Ruochen Zhang2, Melissa Dell3, Benjamin Charles Germain Lee4, Jacob Carlson3, and Weining Li5
    
    1 Allen Institute for AI s
    

```python
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source']
    
    [examples]
    source : ./data/layout-parser-paper.pdf
</pre>

Internally, unstructured creates different "**elements**" for each chunk of text. By default, these are combined, but can be easily separated by specifying `mode="elements"`.

```python
# Create an instance of UnstructuredPDFLoader (mode="elements”)
loader = UnstructuredPDFLoader(FILE_PATH, mode="elements")

# load the data
docs = loader.load()

# print the contents of the document
print(docs[0].page_content)
```

<pre class="custom">1 2 0 2
</pre>

See the full set of element types for this particular article.

```python
set(doc.metadata["category"] for doc in docs) # extract data categories
```




<pre class="custom">{'ListItem', 'NarrativeText', 'Title', 'UncategorizedText'}</pre>



```python
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source', 'coordinates', 'file_directory', 'filename', 'languages', 'last_modified', 'page_number', 'filetype', 'category', 'element_id']
    
    [examples]
    source         : ./data/layout-parser-paper.pdf
    coordinates    : {'points': ((16.34, 213.36), (16.34, 253.36), (36.34, 253.36), (36.34, 213.36)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}
    file_directory : ./data
    filename       : layout-parser-paper.pdf
    languages      : ['eng']
    last_modified  : 2025-01-02T18:23:25
    page_number    : 1
    filetype       : application/pdf
    category       : UncategorizedText
    element_id     : d3ce55f220dfb75891b4394a18bcb973
</pre>

## PyPDFium2

LangChain's [`PyPDFium2Loader`](
https://python.langchain.com/api_reference/community/document_loaders/langchain_community.document_loaders.pdf.PyPDFium2Loader.html) integrates with [PyPDFium2](https://github.com/pypdfium2-team/pypdfium2) to parse PDF documents into LangChain Document objects.

```python
from langchain_community.document_loaders import PyPDFium2Loader

# create an instance of the PyPDFium2 loader
loader = PyPDFium2Loader(FILE_PATH)

# load data
docs = loader.load()

# print the contents of the document
print(docs[10].page_content[:300])
```

<pre class="custom">LayoutParser: A Unified Toolkit for DL-Based DIA 11
    focuses on precision, efficiency, and robustness. The target documents may have
    complicated structures, and may require training multiple layout detection models
    to achieve the optimal accuracy. Light-weight pipelines are built for relatively
    s
</pre>

**Note**: When using `PyPDFium2Loader`, you may notice warning messages related to `get_text_range()`. These warnings are part of the library's internal operations and do not affect the PDF processing
functionality. You can safely proceed with the tutorial despite these warnings, as they are
a normal part of the development environment and do not impact the learning objectives.

```python
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source', 'page']
    
    [examples]
    source : ./data/layout-parser-paper.pdf
    page   : 0
</pre>

## PDFMiner
[PDFMiner](https://github.com/pdfminer/pdfminer.six) is a specialized Python library focused on text extraction and layout analysis from PDF documents.

LangChain's [`PDFMinerLoader`](
https://python.langchain.com/api_reference/community/document_loaders/langchain_community.document_loaders.pdf.PDFMinerLoader.html) integrates with PDFMiner to parse PDF documents into LangChain Document objects.


```python
from langchain_community.document_loaders import PDFMinerLoader

# Create a PDFMiner loader instance
loader = PDFMinerLoader(FILE_PATH)

# load data
docs = loader.load()

# print the contents of the document
print(docs[0].page_content[:300])
```

<pre class="custom">1
    2
    0
    2
    
    n
    u
    J
    
    1
    2
    
    ]
    
    V
    C
    .
    s
    c
    [
    
    2
    v
    8
    4
    3
    5
    1
    .
    3
    0
    1
    2
    :
    v
    i
    X
    r
    a
    
    LayoutParser: A Uniﬁed Toolkit for Deep
    Learning Based Document Image Analysis
    
    Zejiang Shen1 ((cid:0)), Ruochen Zhang2, Melissa Dell3, Benjamin Charles Germain
    Lee4, Jacob Carlson3, and Weining Li5
    
    1 Allen Institute for AI
    s
</pre>

```python
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source']
    
    [examples]
    source : ./data/layout-parser-paper.pdf
</pre>

### Using PDFMiner to generate HTML text

This method allows you to parse the output HTML content through [`BeautifulSoup`](https://www.crummy.com/software/BeautifulSoup/) to get more structured and richer information about font size, page numbers, PDF header/footer, etc. which can help you semantically split the text into sections.

```python
from langchain_community.document_loaders import PDFMinerPDFasHTMLLoader

# create an instance of PDFMinerPDFasHTMLLoader
loader = PDFMinerPDFasHTMLLoader(FILE_PATH)

# load the document
docs = loader.load()

# print the contents of the document
print(docs[0].page_content[:300])
```

<pre class="custom"><html><head>
    <meta http-equiv="Content-Type" content="text/html">
    </head><body>
    <span style="position:absolute; border: gray 1px solid; left:0px; top:50px; width:612px; height:792px;"></span>
    <div style="position:absolute; top:50px;"><a name="1">Page 1</a></div>
    <div style="position:absolute; border
</pre>

```python
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source']
    
    [examples]
    source : ./data/layout-parser-paper.pdf
</pre>

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(docs[0].page_content, "html.parser") # initialize HTML parser
content = soup.find_all("div") # search for all div tags
```

```python
import re

cur_fs = None
cur_text = ""
snippets = []  # collect all snippets of the same font size
for c in content:
    sp = c.find("span")
    if not sp:
        continue
    st = sp.get("style")
    if not st:
        continue
    fs = re.findall("font-size:(\d+)px", st)
    if not fs:
        continue
    fs = int(fs[0])
    if not cur_fs:
        cur_fs = fs
    if fs == cur_fs:
        cur_text += c.text
    else:
        snippets.append((cur_text, cur_fs))
        cur_fs = fs
        cur_text = c.text
snippets.append((cur_text, cur_fs))
# Note: Possibility to add a strategy for removing duplicate snippets (since the header/footer of a PDF appears across multiple pages, it can be considered duplicate information when found)
```

```python
from langchain_core.documents import Document

cur_idx = -1
semantic_snippets = []
# Assumption: headings have higher font size than their respective content
for s in snippets:
    # if current snippet's font size > previous section's heading => it is a new heading
    if (
        not semantic_snippets
        or s[1] > semantic_snippets[cur_idx].metadata["heading_font"]
    ):
        metadata = {"heading": s[0], "content_font": 0, "heading_font": s[1]}
        metadata.update(docs[0].metadata)
        semantic_snippets.append(Document(page_content="", metadata=metadata))
        cur_idx += 1
        continue

    # if current snippet's font size <= previous section's content => content belongs to the same section (one can also create
    if (
        not semantic_snippets[cur_idx].metadata["content_font"]
        or s[1] <= semantic_snippets[cur_idx].metadata["content_font"]
    ):
        semantic_snippets[cur_idx].page_content += s[0]
        semantic_snippets[cur_idx].metadata["content_font"] = max(
            s[1], semantic_snippets[cur_idx].metadata["content_font"]
        )
        continue

    # if current snippet's font size > previous section's content but less than previous section's heading than also make a new
    metadata = {"heading": s[0], "content_font": 0, "heading_font": s[1]}
    metadata.update(docs[0].metadata)
    semantic_snippets.append(Document(page_content="", metadata=metadata))
    cur_idx += 1

print(semantic_snippets[4])
```

<pre class="custom">page_content='Recently, various DL models and datasets have been developed for layout analysis
    tasks. The dhSegment [22] utilizes fully convolutional networks [20] for segmen-
    tation tasks on historical documents. Object detection-based methods like Faster
    R-CNN [28] and Mask R-CNN [12] are used for identifying document elements [38]
    and detecting tables [30, 26]. Most recently, Graph Neural Networks [29] have also
    been used in table detection [27]. However, these models are usually implemented
    individually and there is no uniﬁed framework to load and use such models.
    There has been a surge of interest in creating open-source tools for document
    image processing: a search of document image analysis in Github leads to 5M
    relevant code pieces 6; yet most of them rely on traditional rule-based methods
    or provide limited functionalities. The closest prior research to our work is the
    OCR-D project7, which also tries to build a complete toolkit for DIA. However,
    similar to the platform developed by Neudecker et al. [21], it is designed for
    analyzing historical documents, and provides no supports for recent DL models.
    The DocumentLayoutAnalysis project8 focuses on processing born-digital PDF
    documents via analyzing the stored PDF data. Repositories like DeepLayout9
    and Detectron2-PubLayNet10 are individual deep learning models trained on
    layout analysis datasets without support for the full DIA pipeline. The Document
    Analysis and Exploitation (DAE) platform [15] and the DeepDIVA project [2]
    aim to improve the reproducibility of DIA methods (or DL models), yet they
    are not actively maintained. OCR engines like Tesseract [14], easyOCR11 and
    paddleOCR12 usually do not come with comprehensive functionalities for other
    DIA tasks like layout analysis.
    Recent years have also seen numerous eﬀorts to create libraries for promoting
    reproducibility and reusability in the ﬁeld of DL. Libraries like Dectectron2 [35],
    6 The number shown is obtained by specifying the search type as ‘code’.
    7 https://ocr-d.de/en/about
    8 https://github.com/BobLd/DocumentLayoutAnalysis
    9 https://github.com/leonlulu/DeepLayout
    10 https://github.com/hpanwar08/detectron2
    11 https://github.com/JaidedAI/EasyOCR
    12 https://github.com/PaddlePaddle/PaddleOCR
    4
    Z. Shen et al.
    Fig. 1: The overall architecture of LayoutParser. For an input document image,
    the core LayoutParser library provides a set of oﬀ-the-shelf tools for layout
    detection, OCR, visualization, and storage, backed by a carefully designed layout
    data structure. LayoutParser also supports high level customization via eﬃcient
    layout annotation and model training functions. These improve model accuracy
    on the target samples. The community platform enables the easy sharing of DIA
    models and whole digitization pipelines to promote reusability and reproducibility.
    A collection of detailed documentation, tutorials and exemplar projects make
    LayoutParser easy to learn and use.
    AllenNLP [8] and transformers [34] have provided the community with complete
    DL-based support for developing and deploying models for general computer
    vision and natural language processing problems. LayoutParser, on the other
    hand, specializes speciﬁcally in DIA tasks. LayoutParser is also equipped with a
    community platform inspired by established model hubs such as Torch Hub [23]
    and TensorFlow Hub [1]. It enables the sharing of pretrained models as well as
    full document processing pipelines that are unique to DIA tasks.
    There have been a variety of document data collections to facilitate the
    development of DL models. Some examples include PRImA [3](magazine layouts),
    PubLayNet [38](academic paper layouts), Table Bank [18](tables in academic
    papers), Newspaper Navigator Dataset [16, 17](newspaper ﬁgure layouts) and
    HJDataset [31](historical Japanese document layouts). A spectrum of models
    trained on these datasets are currently available in the LayoutParser model zoo
    to support diﬀerent use cases.
    ' metadata={'heading': '2 Related Work\n', 'content_font': 9, 'heading_font': 11, 'source': './data/layout-parser-paper.pdf'}
</pre>

## PDFPlumber
[PDFPlumber](https://github.com/jsvine/pdfplumber) is a PDF parsing library that excels at extracting text and tables from PDFs.

LangChain's [`PDFPlumberLoader`](
https://python.langchain.com/api_reference/community/document_loaders/langchain_community.document_loaders.pdf.PDFPlumberLoader.html) integrates with PDFPlumber to parse PDF documents into LangChain Document objects.

Like PyMuPDF, the output document contains detailed metadata about the PDF and its pages, and returns one document per page.

```python
from langchain_community.document_loaders import PDFPlumberLoader

# create a PDF document loader instance
loader = PDFPlumberLoader(FILE_PATH)

# load the document
docs = loader.load()

# access the first document data
print(docs[10].page_content[:300])
```

<pre class="custom">LayoutParser: A Unified Toolkit for DL-Based DIA 11
    focuses on precision, efficiency, and robustness. The target documents may have
    complicatedstructures,andmayrequiretrainingmultiplelayoutdetectionmodels
    to achieve the optimal accuracy. Light-weight pipelines are built for relatively
    simple documen
</pre>

```python
show_metadata(docs)
```

<pre class="custom">[metadata]
    ['source', 'file_path', 'page', 'total_pages', 'Author', 'CreationDate', 'Creator', 'Keywords', 'ModDate', 'PTEX.Fullbanner', 'Producer', 'Subject', 'Title', 'Trapped']
    
    [examples]
    source          : ./data/layout-parser-paper.pdf
    file_path       : ./data/layout-parser-paper.pdf
    page            : 0
    total_pages     : 16
    Author          : 
    CreationDate    : D:20210622012710Z
    Creator         : LaTeX with hyperref
    Keywords        : 
    ModDate         : D:20210622012710Z
    PTEX.Fullbanner : This is pdfTeX, Version 3.14159265-2.6-1.40.21 (TeX Live 2020) kpathsea version 6.3.2
    Producer        : pdfTeX-1.40.21
    Subject         : 
    Title           : 
    Trapped         : False
</pre>
