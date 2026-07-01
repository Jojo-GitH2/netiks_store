FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY packages/shared-python /workspace/packages/shared-python
COPY services/admin-service /workspace/services/admin-service

RUN pip install --no-cache-dir /workspace/packages/shared-python /workspace/services/admin-service

WORKDIR /workspace/services/admin-service

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8005"]

