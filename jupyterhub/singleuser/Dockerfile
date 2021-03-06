# Link to all the dockerfiles which this image is based upon. This provides info about various packages
# already present in the image. When updating the upstream image tag, remember to update the links as well.

# Minimal: https://github.com/jupyter/docker-stacks/tree/83ed2c63671f/minimal-notebook/Dockerfile
# Base: https://github.com/jupyter/docker-stacks/tree/83ed2c63671f/base-notebook/Dockerfile

FROM jupyter/datascience-notebook:ubuntu-18.04 as miniconda
# RUN conda update conda python && pip install -U 'pip==18.*'
RUN conda config --set channel_priority strict && \
    conda install --quiet --yes --update-all -c conda-forge \
    'nbconvert' \
    'tqdm' \
    'yapf==0.29*' \
    'rise==5.6.*' \
    'nbdime==2.*' \
    'jupyterhub==1.1.0' \
    'jupyterlab==2.1.*' \
    'ipympl==0.5.8' \
    'nodejs==15.3.0' \
    'jupyter_contrib_nbextensions==0.5*' \
    'jupyter-server-proxy==1.4*' && \
    jupyter labextension install \
    '@jupyter-widgets/jupyterlab-manager' \
    '@jupyterlab/github' \
    'nbdime-jupyterlab' \
    'jupyter-matplotlib@0.7.4' \
    '@jupyterlab/toc' \
    '@jupyterlab/hub-extension' && \
    pip install ipyparallel==6.2.* jupyterlab-github escapism && \
    git clone https://github.com/paalka/nbresuse /tmp/nbresuse && pip install /tmp/nbresuse/ && \
    jupyter serverextension enable --py nbresuse --sys-prefix && \
    jupyter serverextension enable jupyter_server_proxy --sys-prefix && \
    jupyter nbextension install --py nbresuse --sys-prefix && \
    jupyter nbextension enable --py nbresuse --sys-prefix && \
    conda clean  --all -f -y


FROM jupyter/datascience-notebook:ubuntu-18.04

LABEL maintainer Uninett As <system@uninett.no>
USER root

# Setup ENV for Appstore to be picked up
ENV APP_UID=999 \
	APP_GID=999 \
	PKG_JUPYTER_NOTEBOOK_VERSION=6.2.x
RUN groupadd -g "$APP_GID" notebook && \
    useradd -m -s /bin/bash -N -u "$APP_UID" -g notebook notebook && \
    usermod -G users notebook && chmod go+rwx -R "$CONDA_DIR/bin"
COPY --chown=notebook:notebook --from=miniconda $CONDA_DIR $CONDA_DIR
# hadolint ignore=DL3002
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN apt-get update && apt-get install -y --no-install-recommends gnupg2 && \
    curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub | apt-key add - && \
    echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/ /" > /etc/apt/sources.list.d/cuda.list && \
    echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list && \
    rm -rf /var/lib/apt/lists/*

# Refer here for versions https://gitlab.com/nvidia/cuda/blob/ubuntu16.04/10.0/base/Dockerfile,
# note that this uses Ubuntu 16.04.
#
# https://www.tensorflow.org/install/gpu
#  and
# https://github.com/tensorflow/tensorflow/blob/master/tensorflow/tools/dockerfiles/dockerfiles/gpu-jupyter.Dockerfile
# might also be useful for CUDA packages
#
ENV PKG_CUDA_VERSION=10.0.130 \
    NCCL_VERSION=2.4.2 \
    PKG_CUDNN_VERSION=7.5.0.56

RUN apt-get update && apt-get install -y --no-install-recommends \
        "cuda-cudart-10-0=$PKG_CUDA_VERSION-1" \
        "cuda-cublas-10-0=$PKG_CUDA_VERSION-1" \
        "cuda-cublas-dev-10-0=$PKG_CUDA_VERSION-1" \
        "cuda-cudart-dev-10-0=$PKG_CUDA_VERSION-1" \
        "libcudnn7=$PKG_CUDNN_VERSION-1+cuda10.0" \
        "libcudnn7-dev=$PKG_CUDNN_VERSION-1+cuda10.0" \
        "cuda-libraries-10-0=$PKG_CUDA_VERSION-1" \
        "cuda-libraries-dev-10-0=$PKG_CUDA_VERSION-1" \
        "cuda-nvml-dev-10-0=$PKG_CUDA_VERSION-1" \
        "cuda-minimal-build-10-0=$PKG_CUDA_VERSION-1" \
        "cuda-command-line-tools-10-0=$PKG_CUDA_VERSION-1" \
        "libnccl2=$NCCL_VERSION-1+cuda10.0" \
        "libnccl-dev=$NCCL_VERSION-1+cuda10.0" \
        "cuda-compat-10-0=410.48-1" \
        "openmpi-bin=2.1.1-8" && \
    ln -s cuda-10.0 /usr/local/cuda && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
    echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf && \
    echo "/usr/local/cuda/lib64" >> /etc/ld.so.conf.d/nvidia.conf && \
    ln -s /usr/local/cuda/include/* /usr/include/


RUN apt-get update && apt-get install -y --no-install-recommends \
	openssh-client=1:7.6p1-4ubuntu0.3 \
	less=487-0.1 \
	net-tools=1.60+git20161116.90da8a0-1ubuntu1 \
	man-db=2.8.3-2ubuntu0.1 \
	iputils-ping=3:20161105-1ubuntu3 \
	screen=4.6.2-1ubuntu1.1 \
	tmux=2.6-3ubuntu0.2 \
	graphviz=2.40.1-2 \
	cmake \
	rsync=3.1.2-2.1ubuntu1.1 \
	p7zip-full=16.02+dfsg-6 \
	tzdata \
	vim=2:8.0.1453-1ubuntu1.4 \
	unrar=1:5.5.8-1 \
	ca-certificates=20210119~18.04.1 \
    sudo=1.8.21p2-3ubuntu1.4 \
    inkscape=0.92.3-1 \
    "openmpi-bin=2.1.1-8" && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/* && \
    ln -sf /usr/share/zoneinfo/Europe/Oslo /etc/localtime

ENV PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH} \
    LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64:${LD_LIBRARY_PATH} \
    NVIDIA_VISIBLE_DEVICES="" \
    NVIDIA_DRIVER_CAPABILITIES=all \
    TZ="Europe/Oslo"

ENV HOME=/home/notebook \
    XDG_CACHE_HOME=/home/notebook/.cache/
COPY normalize-username.py start-notebook.sh /usr/local/bin/
COPY --chown=notebook:notebook .jupyter/ /opt/.jupyter/
RUN mkdir -p /home/notebook/.ipython/profile_default/security/ && chmod go+rwx -R "$CONDA_DIR/bin" && chown notebook:notebook -R "$CONDA_DIR/bin" "$HOME" && \
    mkdir -p "$CONDA_DIR/.condatmp" && chmod go+rwx "$CONDA_DIR/.condatmp" && chown notebook:notebook "$CONDA_DIR"

# hadolint ignore=DL3002
RUN chmod go+w -R "$HOME" && chmod o+w /home && rm -r /home/notebook

USER notebook
WORKDIR $HOME
CMD ["/usr/local/bin/start-notebook.sh"]
