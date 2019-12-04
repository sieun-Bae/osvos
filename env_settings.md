version: python2, googlecolab (python2, GPU)
* conda as global: export PATH="/Users/username/anaconda/bin:$PATH"

pip3 install Pillow for PIL
pip3 install tqdm

colab setting

1)
from google.colab import drive
drive.mount('/content/gdrive')
import os
import sys
sys.path.insert(0, '/content/gdrive/My Drive/samples/')

2)
add pyflow, pyflowconfig, pyflowwrapper

3)
# coding=utf-8 in file datasets.py


ref) 
os.fdopen method in iOS: https://docs.python.org/3/library/os.html
pyflow? https://illumina.github.io/pyflow/
