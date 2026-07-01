FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY packages/shared-python /workspace/packages/shared-python
COPY services/vendor-service /workspace/services/vendor-service

RUN pip install --no-cache-dir /workspace/packages/shared-python /workspace/services/vendor-service

WORKDIR /workspace/services/vendor-service

CMD ["sh", "-c", "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8002"]
