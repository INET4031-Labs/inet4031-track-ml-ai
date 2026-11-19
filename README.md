# Capstone Track: Machine Learning and AI
 
**Final Sprint | Due before Final Presentations**

> **New to this track? Start here.** The shared [INET 4031 Capstone Guide](https://github.com/INET4031-Labs/inet4031-capstone-guide) explains how to work through any track on your own. Feeling lost in the first few days is normal for a capstone: read the guide's Start Here page, run the Environment Checkpoint below, and take the next small step.
>
> | I need... | Go to |
> |---|---|
> | A reading order and what to do in my first session | [START-HERE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/START-HERE.md) |
> | What I may change in the app, and how to propose my own criterion | [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md) |
> | Local VM: how my k3s and OpenTofu setup differs from this README | [LOCAL-VM](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/LOCAL-VM.md) |
> | What to prioritize in Weeks 11 to 14 | [WEEKLY-PLAN](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/WEEKLY-PLAN.md) and the Sprint Timeline below |
> | Plain-language explanations of this track's concepts | [CONCEPTS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/CONCEPTS.md) |
> | A cheat sheet for each tool in this track | [TOOL-GUIDE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/TOOL-GUIDE.md) |
> | Which file proves which success criterion | [EVIDENCE-CHECKLIST](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/EVIDENCE-CHECKLIST.md) |
> | How the Week 13 rebuild works | [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md) and the Rebuild section below |
> | Help when something is broken | [GETTING-UNSTUCK](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/GETTING-UNSTUCK.md) |
> | How to use AI tools well on this project | [USING-AI](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/USING-AI.md) |
>
> **Local VM students:** much of this README describes the Shared VM setup (k3d, team numbers, `main`, `kubectl apply`). Your setup is a single-node k3s cluster that changes through OpenTofu and your deploy script. Look for the **Local VM** notes in each section, and keep [LOCAL-VM](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/LOCAL-VM.md) open while you work.
 
## Overview
 
Priya Nair, Head of Support Desk at Harborlight Logistics, has a problem: the six agents on the Operations Support Desk spend the first part of every shift reading new incidents in `statustracker` and deciding how urgent each one is and who should handle it. Today every incident is just a title and a status (`open`, `in_progress`, `closed`). There is no category and no severity, so everything looks the same and the loudest ticket gets picked up first. Dana Okafor, Director of Operations, has told Priya that customer promises are slipping on the days when a serious incident sits unread behind ten routine ones.
 
Sam Whitfield, the Platform Team lead, has agreed to let your team try a small machine learning helper: a service that reads an incident title and suggests a category and a severity, and that can flag unusual spikes in incident volume. Marcus Reyes, CTO, has set the rules: no paid services, no API keys, no large model downloads, and everything must run on the existing single server alongside the other teams. This is "small data, small model" work. A classic scikit-learn model that trains in seconds is exactly what is wanted.
 
This is a first exposure to machine learning for many of you. You are not being graded on fancy models. You are being graded on whether you can build a trustworthy, measured, deployed suggestion service that fits inside the cluster. After completing this track, you will have a labeled synthetic dataset, a trained and evaluated model with metrics committed as files, and a containerized inference service running in your k3d cluster with memory limits and its own NetworkPolicy.
 
## The Business Problem
 
**Who is asking:** Priya Nair (Support Desk) with Dana Okafor (staffing) as sponsor. Sam Whitfield is your technical contact. Elena Brandt, Compliance Officer, wants to know that the suggestions are advisory only and that no real customer data is used for training.
 
**Pain today:** Agents triage by hand. The incidents table has `id`, `title`, `status`, `description`, and `created_at`, with no category or severity column, so there is nothing to sort or filter on and no early warning when many incidents arrive at once. `seed.py` gives you titles and statuses only. Your team must create the labels, for example by extending the generator in your own copy, and decide whether to add columns to the database or keep labels in separate files (say which, and why, in `ARCHITECTURE.md`).
 
**Constraints:**
- Budget: no new paid services, no paid APIs, no API keys. Everything runs on the existing single server.
- No downloading pretrained models or large datasets. Train from data your team generates.
- Compliance: training data must be synthetic. Do not copy real employee or customer names into it. Suggestions are advice for a human, never an automatic action.
- Deadline: a working demo before the next quarterly business review, and the contract renewal in six weeks depends on the Support Desk showing measurable improvement.
**Success criteria** (your evidence must be files committed to your repo):
1. A labeled synthetic dataset of at least 5,000 incidents (each with a title, a category from at least 4 categories, and a severity from at least 3 levels) is generated by a script committed under `capstone/ml-ai/`, and the script reproduces the same file when run with the same random seed.
2. A trained model is evaluated on a held-out test set (not used in training) and the metrics file (for example `metrics.json`) is committed. Severity accuracy is at least 0.80 and macro-averaged F1 is reported for both category and severity. The metrics file also records a simple baseline (for example always predicting the most common label) so the improvement is visible.
3. The inference service runs in your k3d cluster as a Deployment with memory requests and limits, and answers a request with a category, a severity, and a confidence value in under 500 ms (record the timing).
4. The service is reachable only as your design intends: your NetworkPolicy allows exactly the traffic you document, and you show evidence (command output saved to a file) that an unauthorized pod is blocked.
5. Your total additions (all pods you add) stay under 512Mi of memory limits combined, and the container image is under 600 MB (record `docker image ls` output).
## Learning Objectives
 
- Create labeled training data from unlabeled records by extending a generator, and explain why labels are the hard part of machine learning
- Train, evaluate, and save a small scikit-learn model, and explain train/test split, accuracy, precision, recall, and F1
- Compare a model against a trivial baseline to decide whether it is actually useful
- Package a model into a small container image and run it as a non-root, memory-limited Kubernetes workload
- Apply NetworkPolicy to a new service in a default-deny cluster
- Describe the limits and risks of automated suggestions (wrong answers, drift, bias in synthetic data)
## Prerequisites
 
Everything from earlier weeks must still work: the `statustracker` Flask API, PostgreSQL, the Docker Compose stack in `week-2/`, your team's k3d cluster, the default-deny NetworkPolicies and their allows, and Prometheus in the `monitoring` namespace.

> **Local VM (Ubuntu, individual):** your prerequisites are Weeks 4 to 10 of your section: the k3s node with `web` and `db` managed by OpenTofu in `week-5/`, Prometheus (Week 6), the Deploy gate and `deploy.sh` (Week 7), resource limits and the `db-clients` NetworkPolicy (Weeks 8 and 9). Your table is `tickets`, not `incidents`, with the same columns, and there is no `seed.py`: you write your generator from scratch under `capstone/ml-ai/`, which criterion 1 asks for anyway.
 
### Environment Checkpoint
 
Run these and save the output to a file in `capstone/ml-ai/evidence/`. Then commit it.
 
```bash
docker compose -f week-2/docker-compose.yml ps
curl -s http://localhost:8080/health
k3d cluster list
kubectl get nodes
kubectl get pods -A
kubectl get networkpolicy
free -h
```
 
Replace `8080` with your Compose port (8080 + team number on the shared VM).

**Local VM:** your Compose stack has been stopped since Week 4, so use this block instead:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get networkpolicy
curl -s http://$(kubectl get service web -o jsonpath='{.spec.clusterIP}'):8080/health
./week-10/sql.sh
tail -5 ~/deploy.log
free -h
```
 
> **Shared VM (RHEL, teams):** Use your own team's ports for every command in this track.
 
> **Local VM (Ubuntu, individual):** There is no team number. Reach the app at the `web` Service address shown above, or in your browser with `kubectl port-forward --address 0.0.0.0 service/web 8080:8080` and http://localhost:8080. To check Prometheus, run `kubectl port-forward --namespace monitoring service/prometheus-server 9090:80` in its own terminal and `./week-6/pq.sh 'up'` the way you did in Week 6.
 
## Team Port Assignments
 
> **Shared VM (RHEL, teams):** Teams share one VM. Your Compose port is 8080 + team number, your Kubernetes port is 8090 + team number, and your Prometheus port is 9090 + team number. This track adds one port: the inference service port-forward, which is **9290 + team number**. Use only your own ports. Do not send traffic to another team's ports.
 
| Team | Compose | Kubernetes | Prometheus | Inference service (new) |
| ---- | ------- | ---------- | ---------- | ----------------------- |
| 1    | 8081    | 8091       | 9091       | 9291                    |
| 2    | 8082    | 8092       | 9092       | 9292                    |
| 3    | 8083    | 8093       | 9093       | 9293                    |
| 4    | 8084    | 8094       | 9094       | 9294                    |
| 5    | 8085    | 8095       | 9095       | 9295                    |
| 6    | 8086    | 8096       | 9096       | 9296                    |
| 7    | 8087    | 8097       | 9097       | 9297                    |
 
> **Local VM (Ubuntu, individual):** There is no team number. The app is on port 8080 (through the `web` Service or your port-forward), Prometheus on 9090, and the inference service port-forward on **9290**. From inside the VM you can also call the inference Service by its cluster address, the same way `week-10/run-k6.sh` reaches `web`. A browser is not needed for this track; use `curl`.
 
The inference service itself listens on a port of your choosing inside the pod (for example 5001). Only the port-forward uses the 9290 + team number pattern. You may instead keep the service reachable only from inside the cluster (for example only from the Flask pods). If you do, say so in `ARCHITECTURE.md` and still document how you tested it.
 
## Tools Available
 
| Tool | What it's for | Verify (both OSs) | Ubuntu install |
| ---- | ------------- | ----------------- | -------------- |
| [Python 3 + venv](https://docs.python.org/3/library/venv.html) | Run the training scripts in a virtualenv | `python3 --version` | `sudo apt-get install -y python3 python3-venv` |
| [scikit-learn](https://scikit-learn.org/stable/) | Small classical ML models (TF-IDF, logistic regression, IsolationForest) | `~/ml-ai-venv/bin/python -c "import sklearn; print(sklearn.__version__)"` | `~/ml-ai-venv/bin/pip install --no-cache-dir scikit-learn` |
| [numpy](https://numpy.org/doc/stable/) and [scipy](https://docs.scipy.org/doc/scipy/) | Numeric support that scikit-learn needs (installed with scikit-learn) | `~/ml-ai-venv/bin/python -c "import numpy, scipy; print(numpy.__version__, scipy.__version__)"` | `~/ml-ai-venv/bin/pip install --no-cache-dir numpy scipy` |
| [joblib](https://joblib.readthedocs.io/) | Save and load the trained model file | `~/ml-ai-venv/bin/python -c "import joblib; print(joblib.__version__)"` | `~/ml-ai-venv/bin/pip install --no-cache-dir joblib` |
| [Flask](https://flask.palletsprojects.com/) | The tiny inference web service (FastAPI is also fine) | `~/ml-ai-venv/bin/python -c "import flask; print(flask.__version__)"` | `~/ml-ai-venv/bin/pip install --no-cache-dir flask` |
| [Docker](https://docs.docker.com/) | Build the inference image | `docker --version` | Already installed in earlier weeks |
| [k3d](https://k3d.io/) and [kubectl](https://kubernetes.io/docs/reference/kubectl/) | Import the image and deploy to your cluster | `k3d version && kubectl version --client` | Already installed in earlier weeks |
| [curl](https://curl.se/docs/) and [jq](https://jqlang.org/manual/) | Call the service and read JSON | `curl --version && jq --version` | `sudo apt-get install -y curl jq` |
 
> **Shared VM (RHEL, teams):** The system tools in this table (everything except Python packages in your own virtualenv) are pre-installed by the TA. Run the Verify command for each tool. If a command is not found, post in the course channel or tell the TA or professor before continuing. Do not install anything system-wide.
 
> **Local VM (Ubuntu, individual):** Your VM runs Ubuntu Server 26.04.1. You install tools yourself. Run the command in the last column, then the Verify command. k3d is not part of your setup: skip its Verify command and use `kubectl get nodes` instead.
 
**Versions.** This table has no version numbers on purpose: use the current stable release of each tool, so these instructions keep working in future semesters. Record what you actually used. Run each Verify command and save the output to `capstone/ml-ai/TOOL-VERSIONS.txt`, then commit it. For Python packages, also save `~/ml-ai-venv/bin/pip freeze > capstone/ml-ai/requirements.txt` after you install them, so a teammate (or a future you) can rebuild the same environment.
 
**Create your virtualenv.** You create this yourself. It lives in your own home directory, so no sudo and no system pip are needed.
 
```bash
python3 -m venv ~/ml-ai-venv
~/ml-ai-venv/bin/pip install --upgrade pip
~/ml-ai-venv/bin/pip install --no-cache-dir scikit-learn numpy scipy joblib flask
```
 
> **Shared VM (RHEL, teams):** Each person who runs the code creates their own virtualenv from the same package list (commit it as `requirements.txt`). Never use `sudo` or system `pip`.
 
> **Local VM (Ubuntu, individual):** Use the virtualenv above. Do not use `pip install` outside it.
 
These packages publish prebuilt wheels, so nothing should compile. If `pip` starts compiling something for more than a minute, stop and ask the TA. Do not install PyTorch, TensorFlow, or Jupyter.
 
> **Why freeze your versions?** A saved model file only loads reliably with the same scikit-learn version that trained it. Because this table does not pin versions, the `requirements.txt` you generate with `pip freeze` is your pin. Build your inference image from that same `requirements.txt` so the training environment and the serving environment match.
 
> **Background:** scikit-learn, numpy, and scipy together make an image of a few hundred MB. That is normal but not free: teams share one disk. Start from a current `python:3.x-slim` base image (choose a Python version your frozen requirements install on), install with `pip --no-cache-dir`, install only what the service needs (it does not need pandas or matplotlib), and keep the saved model file small (a TF-IDF plus logistic regression model is usually well under 5 MB).
 
> **Enterprise Pattern:** Train once, serve many. Training happens offline in a script and produces a versioned model file plus a metrics file. The serving container only loads that file and answers requests. Changing the model means a new image tag, not editing a running pod.
 
## Example Approaches
 
These are examples, not requirements. Combine them, change them, or ignore them.
 
**Approach A: Text classifier for category and severity.** Extend your own copy of `seed.py` so each generated incident also gets a category and a severity chosen by rules you define (for example, titles about certificates or logins map to a security category). Train a TF-IDF vectorizer with a linear classifier, one for category and one for severity, and serve both. This demonstrates the full loop of labeling, training, evaluating, and serving, and shows how label quality limits model quality.
 
> **Local VM:** you have no `seed.py`. Write a generator under `capstone/ml-ai/` that produces titles, categories, and severities, and decide whether it also inserts rows into `tickets` (for example through `./week-10/sql.sh` or a CSV loaded with `psql`).

**Approach B: Volume anomaly alert.** Instead of (or in addition to) classifying text, count incidents per hour from the `created_at` column and train an IsolationForest or a simple rolling threshold to flag unusual spikes. The service accepts a recent count and answers "normal" or "unusual". This demonstrates unsupervised learning, and it connects naturally to the Prometheus metrics you already collect.
 
**Approach C: Honest-evaluation focus.** Build a smaller classifier but invest in the evaluation: a confusion matrix saved as a file, per-class precision and recall, a deliberately harder test set (titles with unseen wording), and a written explanation of where the model should not be trusted. This demonstrates that a measured, modest model is more useful to Priya's team than an unmeasured impressive one.
 
## Track Tips and Common Trip-Ups

These notes come from where students usually get stuck. They do not change the success criteria; they help you reach them. Concepts are explained in [CONCEPTS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/CONCEPTS.md) and commands in [TOOL-GUIDE](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/TOOL-GUIDE.md). If a flag here does not work on your installed version, check the tool's `--help`.

**Labels are the hard part (criteria 1 and 2)**

- **What one labeled row could look like.** Think in columns before code. Each row needs a title, a category (at least 4, chosen to fit a logistics support desk, such as scanners and handhelds, the dispatch app, printers and labels, logins and access, network), and a severity (at least 3 levels, such as low, medium, high). Then write the rules that connect them in plain language first, for example "titles that mention every depot or a full outage are high," "a single printer is low unless it blocks shipping." If you plan Approach B, include `created_at` too. Put that rule list in `ARCHITECTURE.md` before you write the generator: it is your labeling policy, and it is what criterion 2's accuracy actually measures.

- If your generator assigns labels from keywords in the title and the model reads the same titles, it will score close to perfect. That tells you about your generator, not about real incidents. Make the data more realistic and harder: synonyms, titles that could fit two categories, a small share of deliberately "noisy" labels, and a test set with wording the model never saw. Write what you did and why in `ARCHITECTURE.md`. Discussion question 2 asks about exactly this.
- Seed everything: Python's `random`, NumPy, and `random_state=` in `train_test_split` and the model. Prove reproducibility by running the generator twice and comparing `sha256sum` of both outputs.

**Measuring honestly (criterion 2)**

- Train your baseline first: `DummyClassifier(strategy="most_frequent")` from `sklearn.dummy`. Record its accuracy and macro F1 in the same metrics file as your model.
- Accuracy alone can hide a problem. If 70% of incidents are "Low," a model that always says "Low" scores 0.70 while missing every Critical incident. That is why the criterion also asks for macro F1, which weights each class equally.
- A `Pipeline` of `TfidfVectorizer` and `LogisticRegression` keeps the text processing and the model in one file you can save with joblib. One pipeline per target (category, severity) is simplest.
- For the confidence value, `predict_proba` gives a probability per class; the highest one is a reasonable confidence to return.

**Serving and proving it (criteria 3 to 5)**

- **Decide the contract before the code.** Write down what a request and a response look like, then build to it and test it in your QA file. For example:

  ```text
  POST /predict
  request:  {"title": "Handheld scanner at Depot 4 will not sync"}
  response: {"category": "scanners", "severity": "medium", "confidence": 0.82, "advisory": true}
  ```

  The names and values are only an example. Decide whether you return one confidence or one per target, what an empty or missing title returns (an HTTP 400 with a message is common), and how the response makes clear the suggestion is advisory, which Elena asked for. Put the contract in `ARCHITECTURE.md`.

- Load the model once when the service starts, not on every request, or your timing will suffer. Measure with `curl -s -o /dev/null -w "%{time_total}\n" ...` and save several runs, not one.
- For the blocked-traffic test, use a short-lived pod with a memory limit (see the debug pod example below) that does not match your allow policy, and request your service's cluster address. A request that times out is the expected result; save the output.
- Check image size with `docker image ls ml-ai-infer:team<N>`. If it is too large, look at the base image and remove packages the service does not import.

Debug pod with a memory limit (`kubectl run` cannot set limits in current versions):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: blocked-client
  labels:
    app: blocked-client
spec:
  restartPolicy: Never
  containers:
    - name: client
      image: busybox
      command: ["sleep", "600"]
      resources:
        requests:
          memory: "16Mi"
        limits:
          memory: "64Mi"
```

Then `kubectl exec blocked-client -- wget -T 5 -qO- http://<service>:<port>/<path>`, save the output, and `kubectl delete pod blocked-client`.

**Local VM notes (criteria 3 to 5)**

- **Ship the service through OpenTofu.** Your inference Deployment, Service, and NetworkPolicy go in a new file such as `week-5/ml-infer.tf`, using `kubernetes_deployment_v1`, `kubernetes_service_v1`, and `kubernetes_network_policy_v1`. Follow the pattern of your hardened `web.tf` from Week 8: non-root user, `allow_privilege_escalation = false`, CPU and memory requests and limits, and `image_pull_policy = "IfNotPresent"`. The Deploy gate's security scan fails without them.
- **Import before you merge.** Build `ml-ai-infer:v1`, then `docker save ml-ai-infer:v1 -o /tmp/infer.tar && sudo k3s ctr images import /tmp/infer.tar` before the pull request that deploys it. Check with `sudo k3s crictl images | grep ml-ai-infer`. Image size comes from `docker image ls ml-ai-infer:v1`.
- **There is no default-deny on your VM.** Your inference pod accepts traffic from every pod until you add a policy that selects it. Once your policy selects it, only what the policy allows gets in, which is exactly what criterion 4 needs. Allow from `app: web` (your Flask pods are labeled `web`, not `flask`), and translate the YAML fragment in Rules of the Road into a `kubernetes_network_policy_v1` resource.
- **Calling it from the app is optional and fits Make It Yours.** If `web` calls the inference service when a ticket is created, add the new setting (the service URL) to your ConfigMap in `week-5/` rather than hard-coding it.

## Rules of the Road
 
- **Obey the shared-VM limits.** About six teams share one VM. Keep all your additions under **512Mi** of memory limits combined. Every pod you deploy needs `resources.requests` and `resources.limits` for memory. Check with `kubectl describe pod <name>` (look at Requests and Limits), `free -h`, and `kubectl top pods -A`. If `kubectl top` errors, tell the TA.
- **Training is CPU-only and short.** Training should finish in seconds to a few minutes. No GPU tooling, no PyTorch or TensorFlow, no Spark, no always-on JupyterLab, no downloading pretrained models.
- **No paid APIs or API keys.** Do not call any hosted AI service and do not put keys in the repo.
- **Synthetic data only.** Labels come from your own generator. Do not paste real names or customer data into the training set.
- **Builds use host networking.** Containers on the default Docker network cannot reach the internet from the campus network, so build with `docker build --network host -t ml-ai-infer:team<N> capstone/ml-ai/service`. Do not use `build: network: host` in Compose, do not disable firewalld, and do not edit `/etc/docker/daemon.json`.
- **Tag with your team number.** All teams share one Docker daemon. Tag every image `<name>:team<N>` and import it with `k3d image import ml-ai-infer:team<N> -c <your cluster name>`. Never use `-c myapp`. Your cluster was created with `--subnet 100.97.<team>.0/24` so pods can pull images; do not recreate it during the sprint. For the Week 13 rebuild, your RUNBOOK must include the exact `k3d cluster create` command with every flag (on the Shared VM, keep the same `--subnet 100.97.<team>.0/24` with your team number; on the Local VM, use the same flags you used the first time).
- **Non-root Dockerfile and file modes.** The VM uses `umask 077`, so files you edit are mode 600 and a non-root container user cannot read them. In your Dockerfile use `COPY --chmod=644` for the files the service reads (code and model file), or run `chmod 644` on them before building.
- **New pods need a NetworkPolicy allow.** Your cluster has default-deny ingress in `default`, so your inference pod will be blocked from receiving traffic from other pods until you add an allow policy. A fragment to adapt (names and labels are yours to choose):
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-flask-to-infer
spec:
  podSelector:
    matchLabels:
      app: ml-infer
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: flask
      ports:
        - protocol: TCP
          port: 5001
```
 
  Decide who is allowed to call the service and write only that allow. Then test that a pod that is not allowed gets blocked, and confirm that `kubectl port-forward` to your service still works with the policy in place (port-forward is not blocked by NetworkPolicy, the same way your Week 5 Prometheus and app port-forwards work).
- **Port-forward in the foreground.** Run `kubectl port-forward` in its own terminal and leave it there. Never add `&`. If a stale one is left behind, run `pkill -u $USER -f "kubectl port-forward"`.
- **No sudo for cluster tools.** Run `k3d`, `kubectl`, and `helm` without `sudo`. On the shared VM you never need `sudo`.
- **Stay inside your own cluster.** Do not scan or probe the VM, the campus network, or other teams' ports. Test only your own service on your own ports.
- **Git rules (RHEL).** Work on a branch: `git switch -c <branch>`, `git push -u origin <branch>`, open a pull request, wait for build-and-push, then merge. Never push to `main`.
- **Keep files small.** The generated dataset and model should be small enough to review in a PR (a few MB at most).

### Local VM (Ubuntu, individual) rules

The rules above that mention other teams, team numbers, k3d, `--network host`, firewalld, `/etc/docker/daemon.json`, `umask 077`, or [PREDICT ONLY] describe the Shared VM. On your own VM these apply instead:

- **Change the cluster through OpenTofu.** Anything that should last goes in `week-5/*.tf`, through a pull request, the Deploy gate, and `week-7/deploy.sh`. Throwaway test pods may use `kubectl apply`, and you delete them afterward. A change made with `kubectl` to an object OpenTofu manages is undone by the next deploy.
- **The gate checks your additions.** Every container you add in `week-5/` needs CPU and memory requests and limits and must not allow privilege escalation, or the `security scan` job fails and the ruleset blocks the merge.
- **New image, new tag.** Build a new tag (for example `:v1`, `:v2`), import it with `docker save` and `sudo k3s ctr images import` **before** the pull request that uses it, and never overwrite the tag that is running.
- **`master` and `protect-master` stay.** Work on branches and merge through pull requests. Do not delete the ruleset, remove its checks, or add a bypass.
- **No default-deny.** Only the `db` pods are protected (by `db-clients`). A pod you add accepts traffic from anywhere until a NetworkPolicy selects it.
- **sudo** is for package installs, `sudo install` of single binaries, and `sudo k3s ctr`. Run `kubectl`, `helm`, `tofu`, and `k6` as your normal user.
- **Memory budget still applies:** keep your additions under 512Mi of memory limits combined. Check `free -h` before and after.
- **Image tags:** use versions such as `ml-ai-infer:v1`, not `team<N>`, and no `-c` flag (there is no k3d).

## How to Run This Sprint
 
### Sprint Timeline (Weeks 11 to 14)

Both sections follow the same four weeks. The planning steps below still apply; this table says where your work should be at the end of each week. The full plan is in [WEEKLY-PLAN](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/WEEKLY-PLAN.md).

| Week | By the end of the week |
|---|---|
| 11 Kickoff | Environment Checkpoint saved. Generator with labels and a fixed seed producing at least 5,000 rows. A first metrics file that includes the baseline, even if the model is weak. QA test cases drafted. |
| 12 Build | Trained models and `metrics.json` compared to the baseline. Service built into an image, running as a Deployment with memory limits, behind its NetworkPolicy. First timing recorded. |
| 13 Consolidate and rebuild | Blocked-traffic evidence, image size evidence, one documented wrong prediction, RUNBOOK and ARCHITECTURE, rebuild, and gap log. |
| 14 Present | Rebuild presentation (live on the Shared VM, recorded on the Local VM) plus your metrics and a live request to the service. |

### Shared VM (RHEL, teams)
 
1. **Sprint planning.** As a team, read the success criteria and turn them into GitHub issues on your project board. Split big items: data generation, labeling rules, training, evaluation, Dockerfile, manifests, NetworkPolicy, docs.
2. **Suggested roles.**
   - **Scrum Master:** runs check-ins, keeps the board current, chases blockers, owns the retrospective.
   - **System Admin:** owns the Dockerfile, image build and import, Deployment, resource limits, NetworkPolicy, and port-forward.
   - **Developers (2):** one owns data generation and labeling rules; one owns training, evaluation, and the Flask service.
   - **QA:** writes test cases in `docs/qa-report-final.md` and runs them against the service and the metrics.
3. **QA first.** QA writes the test cases in `docs/qa-report-final.md` before the implementation is done, so the team knows what "working" means.
4. **Mid-sprint check-in.** Halfway through, the team shows the dataset file, a first metrics file (even a weak one), and a built image. Adjust scope if the NetworkPolicy or memory limits are causing trouble.
5. **Merge through pull requests.** Never push to `main`.
6. **Retrospective.** At the end, fill out `docs/final-sprint-retrospective.md` together.
### Local VM (Ubuntu, individual)
 
You follow the same Sprint Timeline as the Shared VM teams, alone.

- **Week 11:** Environment Checkpoint saved. Generator with labels and a fixed seed producing at least 5,000 rows. A first metrics file that includes the baseline, even if the model is weak. Your test cases drafted in `docs/qa-report-final.md`.
- **Week 12:** trained models and `metrics.json` compared to the baseline. Service built into an image, imported into k3s, and running through `week-5/` as a Deployment with memory limits, behind its NetworkPolicy. First timing recorded.
- **Week 13:** blocked-traffic evidence, image size evidence, one documented wrong prediction, RUNBOOK, ARCHITECTURE, and the top-level README section, then rebuild on a new VirtualBox VM and keep a gap log (see the Rebuild section).
- **Week 14:** recorded presentation of your rebuild.
- You write your own test cases in the same QA file and sign as both author and tester. A short reflection replaces the retrospective.

## Rebuild (Weeks 13 and 14)

Your work has to come back up from your repository alone. Start from a clean VM, clone your repo, and follow only your README, RUNBOOK, and scripts. Every step that fails or that your README forgot is a gap: write it in `capstone/ml-ai/REBUILD-GAPS.md`, fix it by pull request, and update the README. The full contract, the "what lives outside git" checklist, and the gap log template are in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md).

> **Shared VM (RHEL, teams):** In Week 13 your team takes a turn in a queue to test the rebuild on a separate VM, which is wiped between teams. In Week 14 you present live: the tools are pre-installed, and your team clones the repo and brings the app up from your README and setup instructions. There is no sudo on the rebuild or presentation VM, so every step your README asks someone to run must work without it. Your Ansible playbooks and any [PREDICT ONLY] code stay reference-only: do not run them, and make sure bringing the app up does not depend on them. Recreate your k3d cluster with the same `--subnet 100.97.<team>.0/24` flag you used before.

> **Local VM (Ubuntu, individual):** In Week 13 create a new VM in VirtualBox using the same steps you used for your original VM, clone your repo, and follow your README. Keep the original VM until the rebuild works. In Week 14 you present a recording of the rebuild. Start from the "Rebuild on a fresh VM" sections you wrote in your Week 7, 8, and 9 READMEs (k3s, OpenTofu variables, image import, deploy and backup cron lines), then add everything this track added. The full Local VM order is in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md). You have sudo on your own VM; every step that needs it must be in your README.

**Proof of life for this track:** the generator reproduces the same dataset from the seed (`sha256sum` matches); the model loads; one request returns a category, a severity, and a confidence.

**Gaps this track usually finds:**
- The `k3d cluster create` command and its `--subnet` flag are not in the repo.
- The model file was trained on the old VM and never committed, and the training command is not documented.
- The image was built and imported by hand, so the build and `k3d image import` commands are missing from the RUNBOOK.
- Local VM: the inference image was imported into k3s on the old VM only, so the first deploy on the new VM cannot start the pod. The `docker save` and `sudo k3s ctr images import` steps belong in the README before `./week-7/deploy.sh`.

## Deliverables
 
- Implementation committed under `capstone/ml-ai/` in your repo (merged via PR on the RHEL section)
- `capstone/ml-ai/RUNBOOK.md`: how to operate, check, and recover the implementation
- `capstone/ml-ai/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages): the tool versions you actually used (see Tools Available)
- `capstone/ml-ai/ARCHITECTURE.md` with an architecture diagram (Mermaid OK) showing the VM, cluster, app, and new components
- A "How to run this" section added to the repo's top-level README so someone who clones the repo can bring the track implementation up
- `docs/qa-report-final.md` with your own test cases and results
- `capstone/ml-ai/REBUILD-GAPS.md`: the gap log from your Week 13 rebuild (template in [REBUILD](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/REBUILD.md))
- Shared VM: `docs/final-sprint-retrospective.md`. Local VM: a short reflection (the "Individual Reflection" section of the retrospective file)
- Final presentation/demo: 8-10 minutes for a team on RHEL, 5 minutes for an individual. The RHEL demo is CLI-driven. Show the dataset and metrics files, the service answering a request, the memory limits and NetworkPolicy, and one case where the model is wrong and what you learned from it.
- Week 14 rebuild presentation: live on the Shared VM (clone the repo and bring the app up from your README), recorded on the Local VM. See the Rebuild section.
Track-specific items:
- The dataset generator script and the dataset file (or instructions to regenerate it), with a fixed random seed
- The training and evaluation scripts, plus the committed metrics file(s) including the baseline comparison
- The saved model file produced by joblib (small), or a documented command that recreates it
- `capstone/ml-ai/service/` with the Dockerfile, `requirements.txt`, and service code
- Kubernetes manifests for the Deployment, Service, and NetworkPolicy
- An `evidence/` folder: `kubectl describe` output showing requests/limits, `docker image ls` output, one blocked-traffic test, and one request/response timing
## Discussion (add to Google Doc)
 
1. Priya's agents will see a suggested severity. What could go wrong if they trust it too much? What would you put in the response (for example a confidence value) to help them decide?
2. Your labels came from rules you wrote. What does the model's test accuracy actually tell you, and what does it not tell you about real incidents?
3. Compare your model's score to the baseline. Is the difference worth the cost of running another pod on a shared server? How would you explain that to Marcus Reyes?
4. Elena Brandt asks whether a wrong suggestion could be a compliance problem. What controls (advisory only, logging, human review) would you recommend?
5. After the contract renewal, incidents change (new systems, new wording). How would you notice the model getting worse, and what would retraining involve?
## Getting Help
 
Clarifications about this track go to the TA or the professor. There is no wiki for the capstone tracks. If a tool is missing or `kubectl top` errors on the shared VM, tell the TA rather than working around it.

**Before you ask:** work through [GETTING-UNSTUCK](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/GETTING-UNSTUCK.md) for about 30 minutes, then ask with the command, its full output, what you expected, and what you tried.

**Using AI:** AI assistants are welcome as a tool for learning how these tools work and how enterprises use them. Do not rely on them to produce your work: understand every line you commit, verify AI answers against the tool's documentation, never paste AI output into evidence files, and never paste secrets into an AI tool. See [USING-AI](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/USING-AI.md).
