FROM python:3.9

ARG MODEL_VERSION
ARG REPO
ARG BRANCH

WORKDIR /model-test

#It is better to get model, dataset and repo now than download it every time when pipeline starts
#I suppose it is all the dependencies that test has to have

# venv can be used for dependencies installation but it is a container
RUN python3 -m pip install -r ./requirements.txt

#Put the model and testing software onto container
RUN curl "https://web.share.local/${MODEL_VERSION}" --silent --output "./model/${MODEL_VERSION}" \
    && git clone -b "${BRANCH}" "${REPO}" ./test

#Put the dataset onto container
COPY ./dataset .

#Run the script as the entrypoint
ENTRYPOINT [ "/bin/bash", "./test-model.sh" ]