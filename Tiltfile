# -*- mode: Python -*-

"""
* Frontend
  * Language: Go
  * Other notes: calls out to both `letters` and `numbers` microservices.
* Letters
  * Language: JavaScript
  * Other notes: Uses yarn. Does a `yarn install` for package dependencies iff they have changed.
* Numbers
    * Language: Python
    * Other notes: does a `pip install` for package dependencies. Reinstalls dependencies iff they have changed.
"""

k8s_yaml([
    'fe/deployments/fe.yaml',
    'letters/deployments/letters.yaml',
    'numbers/deployments/numbers.yaml',
    'db_writer/deployments/db_writer.yaml',
])

# Port-forward your frontend so you can hit it locally -- you can access
# the 'fe' service in your browser at http://localhost:8000
k8s_resource('fe', port_forwards='8000')
k8s_resource('letters', port_forwards='8001')
k8s_resource('numbers', port_forwards='8002')
k8s_resource('db-writer', port_forwards='8003')

# For all services, tell Tilt how to build the docker image, and how to Live Update
# that service -- i.e. how to update a running container in place for faster iteration.
# See docs: https://docs.tilt.dev/live_update_tutorial.html

# Service: fe
docker_build('abc123/fe', 'fe',  # ~equivalent to: docker build -t abc123/fe ./fe
             live_update=[
                 sync('./fe', '/go/src/github.com/windmilleng/abc123/fe'),
                 run('go install github.com/windmilleng/abc123/fe'),
                 restart_container()
             ])

# Service: letters
docker_build('abc123/letters', 'letters',
             live_update=[
                 sync('./letters/src', '/app'),
                 sync('./letters/package.json', '/app/package.json'),
                 sync('./letters/yarn.lock', '/app/yarn.lock'),
                 # run `yarn install` IF `package.json` or `yarn.lock` has changed
                 run('cd /app && yarn install', trigger=['./letters/package.json', './letters/yarn.lock']),
                 restart_container(),
             ])

# Service: numbers
docker_build('abc123/numbers', 'numbers',
             live_update=[
                 sync('./numbers', '/app'),
                 # run `pip install` IF `requirements.txt` has changed
                 run('cd /app && pip install -r requirements.txt', trigger='numbers/requirements.txt'),
             ])

# Service: recorder
docker_build('abc123/db_writer', 'db_writer',
             live_update=[
                 sync('./db_writer', '/go/src/github.com/windmilleng/abc123/db_writer'),
                 run('go install github.com/windmilleng/abc123/db_writer'),
                 restart_container()
             ])
