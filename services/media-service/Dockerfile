FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY packages/shared-python /workspace/packages/shared-python
COPY services/media-service /workspace/services/media-service

RUN pip install --no-cache-dir /workspace/packages/shared-python /workspace/services/media-service

WORKDIR /workspace/services/media-service

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8004"]

