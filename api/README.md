## Quick Start

### Start the development server

```bash
 uv run uvicorn src.api.main:app
```

Visit http://localhost:8000

### Docker 

```bash 
docker build -t aterminal-api . 

docker run -d -p 127.0.0.1:8000:8000 aterminal-api
```