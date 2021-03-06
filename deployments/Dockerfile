FROM node:8.15.1-jessie AS BUILD
# use multi-stage build to build frontend javascript
WORKDIR /codimd

COPY . ./

RUN yarn install --non-interactive --pure-lockfile && \
    yarn build

# ----------------------------------------------------
# Runtime Stage
FROM node:8.15.1 AS RUNTIME

# build for production
ENV NODE_ENV production
ENV PATH="/home/codimd/.npm-global/bin:${PATH}"

# setup isolated user for more security
ARG USER_NAME=codimd
ARG UID=1500
ARG GID=1500

RUN set +x -ue && \
    wget https://github.com/hackmdio/portchecker/releases/download/v1.0.1/portchecker-linux-amd64.tar.gz && \
    tar xvf portchecker-linux-amd64.tar.gz -C /usr/local/bin && \
    mv /usr/local/bin/portchecker-linux-amd64 /usr/local/bin/pcheck && \
    # Add user and groupd
    groupadd --gid $GID $USER_NAME && \
    useradd --uid $UID --gid $USER_NAME --no-log-init --create-home $USER_NAME && \
    # setup local npm global directory
    mkdir /home/codimd/.npm-global && \
    echo "prefix=/home/codimd/.npm-global/" > /home/codimd/.npmrc && \
    # setup app dir
    mkdir /codimd && \
    # adjust permission
    chown -R $USER_NAME:$USER_NAME /home/codimd

# Copy build stage file to runtime
COPY --from=BUILD /codimd /codimd
RUN chown -R $USER_NAME:$USER_NAME /codimd

# change running user name
USER $USER_NAME
# build project
WORKDIR /codimd

RUN set +x -ue && \
    cliVer=$(cat package.json | grep sequelize-cli | awk '{print substr($1, 2, length($1) - 3)"@"substr($2, 2, length($2) - 3)}') && \
    npm -g install "$cliVer" && \
    yarn install --production --non-interactive --pure-lockfile && \
    yarn cache clean

VOLUME /codimd/public/uploads
EXPOSE 3000

ENTRYPOINT ["/codimd/docker-entrypoint.sh"]
