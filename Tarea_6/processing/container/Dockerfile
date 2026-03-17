FROM python:3.11-slim

WORKDIR /opt/program

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir pandas numpy scikit-learn

ENV PYTHONUNBUFFERED=1

ENTRYPOINT ["python3"]