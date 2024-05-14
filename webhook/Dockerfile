FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Expose the port the webhook listens on
EXPOSE 8000

CMD ["python", "controller.py"]
