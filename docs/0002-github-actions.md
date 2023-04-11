## GitHub Action:

#### Step-1:

Create a config file at `.github/workflows/checks.yml` > Set Trigger > Add steps for running testing and linting

Configure Docker Hub Auth.

```python
---
name: Checks

on: [push]

jobs:
  test-lint:
    name: Test and Lint
    runs-on: ubuntu-20.04
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Checkout
        uses: actions/checkout@v2
      - name: Test
        run: docker-compose run --rm app sh -c "python manage.py test"
      - name: Lint
        run: docker-compose run --rm app sh -c "flake8"
```

#### Step:2

> git add .
> git commit -m "update github action set up to github project"
> git push -u origin main
> go to github project> action>test
