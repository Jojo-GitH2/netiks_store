FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY packages/shared-python /workspace/packages/shared-python
COPY apps/gateway /workspace/apps/gateway

RUN pip install --no-cache-dir /workspace/packages/shared-python /workspace/apps/gateway

WORKDIR /workspace/apps/gateway

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

