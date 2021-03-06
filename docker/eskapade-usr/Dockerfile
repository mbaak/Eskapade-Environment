# --- base image: latest known Eskapade environment
FROM kave/eskapade-base:1.0.0

# --- me
MAINTAINER Max Baak <maxbaak@gmail.com>

# --- switch to root user
USER root

# --- link sh to bash
SHELL ["/bin/bash", "-c"]

# --- useful Ubuntu packages, e.g. pdflatex. 
RUN apt-get update && apt-get install -y --no-install-recommends manpages manpages-dev less vim aspell dvipng && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# --- import pyarrow 0.11.1 fails out of the box; required for feather
#     see bug: https://github.com/apache/arrow/issues/2213
#     downgrade pyarrow to working verion
RUN conda install -y boost-cpp=1.68.0=h3a22d5f_0 && \
    conda install -y parquet-cpp=1.5.1=2 && \
    conda install -y pyarrow=0.11.1=py36hfc679d8_0

# --- trigger automatic installation of required Spark dependencies. Requires Spark setup.
ENV PATH=/usr/local/spark/bin:$PATH
RUN echo "print('Hello world!')" > /tmp/test.py \
    && /usr/local/spark/bin/spark-submit --packages org.diana-hep:histogrammar-sparksql_2.11:1.0.4 /tmp/test.py \
    && rm /tmp/test.py

# --- append sourcing of ROOT env to bashrc
COPY bashrc_appendum /tmp/
RUN cat /tmp/bashrc_appendum >> "${HOME}"/.bashrc && rm /tmp/bashrc_appendum

# --- switch to jovyan user
USER $NB_UID

# --- install other useful python packages
RUN pip install sphinx_rtd_theme==0.4.1 \
    && pip install pymongo==3.4.0 \
    && pip install jaydebeapi==1.1.1 \
    && pip install django==2.0.5 \
    && pip install findspark==1.3.0 \
    && pip install dash==0.28.7 \
    && pip install dash-core-components==0.38.0 \
    && pip install dash-html-components==0.13.2 \
    && pip install dash-renderer==0.14.3 \
    && pip install dash-table==3.1.5

# --- install Eskapade
ENV ROOT_ENV_SCRIPT "/opt/root/bin/thisroot.sh"
RUN source "${ROOT_ENV_SCRIPT}" \
    && pip install Eskapade-Core==1.0.0 \
    && pip install Eskapade==1.0.0 \
    && pip install -e git+https://github.com/KaveIO/Eskapade-ROOT.git@v0.9.0#egg=eskapade-root \
    && pip install Eskapade-Spark==1.0.0

# --- Note on Eskapade-ROOT: pip does not pick up custom cxx installation from pypi repo, 
#     but does so from git when installing in editable mode. Strange but true.
#     See: https://stackoverflow.com/questions/19569557/pip-not-picking-up-a-custom-install-cmdclass
