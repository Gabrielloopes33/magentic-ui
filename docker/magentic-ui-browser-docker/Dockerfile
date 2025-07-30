FROM node:lts-slim

# Evita prompts interativos do apt
ENV DEBIAN_FRONTEND=noninteractive
# Define porta padrão para o Playwright WebSocket
ENV PLAYWRIGHT_PORT=37367

# Instala dependências do SO
RUN apt-get update && \
    apt-get install -y \
      git \
      xvfb \
      x11vnc \
      openbox \
      supervisor \
      python3 \
      python3-pip \
      novnc \
      websockify \
      procps \
      xdg-utils \
      python3-xdg \
      x11-xserver-utils \
      --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*

# Clona noVNC e websockify
RUN git clone --depth 1 --branch v1.6.0 \
      https://github.com/novnc/noVNC.git /usr/local/novnc && \
    git clone --depth 1 --branch v0.13.0 \
      https://github.com/novnc/websockify \
      /usr/local/novnc/utils/websockify

# Diretório de trabalho para a sua aplicação Node
WORKDIR /app

# 1) Copia package.json e instala dependências
COPY docker/magentic-ui-browser-docker/package.json \
     docker/magentic-ui-browser-docker/package-lock.json* ./
RUN npm install

# 2) Instala o Chromium via Playwright
RUN npx playwright@1.51 install --with-deps chromium

# 3) Copia configuração do supervisor
COPY docker/magentic-ui-browser-docker/supervisord.conf \
     /etc/supervisor/conf.d/supervisord.conf

# 4) Copia scripts da aplicação e torna executáveis
COPY docker/magentic-ui-browser-docker/start.sh ./start.sh
COPY docker/magentic-ui-browser-docker/playwright-server.js ./playwright-server.js
COPY docker/magentic-ui-browser-docker/x11-setup.sh ./x11-setup.sh
RUN chmod +x ./start.sh ./x11-setup.sh

# 5) Configuração do Openbox
RUN mkdir -p /root/.config/openbox
COPY docker/magentic-ui-browser-docker/openbox-rc.xml \
     /root/.config/openbox/rc.xml

# 6) Expor portas para noVNC e Playwright
EXPOSE 6080 37367

# Variáveis de ambiente
ENV PLAYWRIGHT_WS_PATH="default"
ENV DISPLAY=":99"

# 7) Workspace para captura de screenshots, logs, etc.
RUN mkdir -p /workspace
WORKDIR /workspace

# 8) (Opcional) Mantém o entrypoint caso queira ainda executar algo antes
# COPY docker/magentic-ui-browser-docker/entrypoint.sh \
#      /usr/local/bin/entrypoint.sh
# RUN chmod +x /usr/local/bin/entrypoint.sh

# Metadados
LABEL org.opencontainers.image.source="https://github.com/Gabrielloopes33/magentic-ui"
LABEL org.opencontainers.image.description="Magentic UI Browser Docker container with Playwright and noVNC"
LABEL org.opencontainers.image.licenses="MIT"

# Faz do supervisord o processo principal em primeiro plano
ENTRYPOINT ["supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf", "-n"]
