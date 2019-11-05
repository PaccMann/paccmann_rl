FROM drugilsberg/rdkit-ubuntu:latest
RUN apt-get update && apt-get install -y git
WORKDIR /paccmann_rl
# install requirements
COPY .travis/requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt
CMD /bin/bash
