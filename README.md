# JupyterHub Templates

The original upstream templates can be found here: https://github.com/jupyterhub/jupyterhub/tree/main/share/jupyterhub/templates

Some other examples of custom templates:

* https://github.com/pangeo-data/pangeo-custom-jupyterhub-templates
* https://github.com/LibreTexts/jupyterhub-templates.git

Discussion of the approach
* https://discourse.jupyter.org/t/customizing-jupyterhub-on-kubernetes/1769/4
* [Video of me walking through the approach](https://youtu.be/o8venP6a8D0)
* Credits: [Kenan Erdogan](https://discourse.jupyter.org/u/bitnik/summary) posted this solution to the Juptyer Discourse channel [here](https://discourse.jupyter.org/t/customizing-jupyterhub-on-kubernetes/1769/3) and LibreTexts posted [instructions](https://github.com/LibreTexts/jupyterhub-templates) to their jupyter-templates repo. Note there are errors in the bit about the directory structure on there repo instructions that are fixed below.


## Directory structure

The JupyterHub is being launched by the command `jupyterhub` and the files for this are under `/usr/local/share/jupyterhub` (normally). The command `static_url()` will take you to `/usr/local/share/jupyterhub/static` (normally). You can specify where `jupyterhub` should look for extra templates but cannot do this for the assets (images and css normally) so you need to put these where they can be found. So you mount your extra assets at `/usr/local/share/jupyterhub/static/extra-assets` and then you refer to them in your templates with something like `href="{{ static_url('extra-assets/css/login.css') }}"`.

## Making Changes

1. Make your changes to the GitHub repo with your templates.
1. Restart the pod that is running the JupyterHub (see below for instructions).

```
kubectl get pods -n dhub
```
Shows you the pods. Your hub pod with be something like `hub-xxxxx`.

To restart it, you can do one of these. I do the first because the 2nd frightens me.

* make a change to config.yaml and then do a helm upgrade.
* delete the pod with
```
kubectl delete pod hub-xxxx -n dhub
```

Debugging if your files are getting into the pod
```
kubectl get pods -n dhub 
# to find the hub pod name
kubectl exec --stdin --tty hub-xxxx -n dhub -- /bin/bash
```
Gets you into a terminal in your pod so you can look around.

## Edits to your JupyterHub config file.

Add to the Z2JH config file. Uses the `alpine/git` image as an initContainer to clone the repo and move the template files to the right places.
```yaml
  hub:
    initContainers:
      - name: git-clone-templates
        image: alpine/git
        command:
          - /bin/sh
          - -c
        args:
          - >-
              git clone --branch=master https://github.com/nmfs-opensci/jupyterhub-templates.git &&
              cp -r jupyterhub-templates/templates/* /templates &&
              cp -r jupyterhub-templates/extra-assets/* /extra-assets
        volumeMounts:
          - name: custom-templates
            mountPath: /templates
          - name: custom-templates-extra-assets
            mountPath: /extra-assets
    extraVolumes:
      - name: custom-templates
        emptyDir: {}
      - name: custom-templates-extra-assets
        emptyDir: {}
    extraVolumeMounts:
      - name: custom-templates
        mountPath: /usr/local/share/jupyterhub/custom_templates
      - name: custom-templates-static
        mountPath: /usr/local/share/jupyterhub/static/extra_assets
    extraConfig:
      templates: |
        c.JupyterHub.template_paths = ['/usr/local/share/jupyterhub/custom_templates/']
```

In your templates, you will refer to items from extra_assets with things like this
```
src="{{static_url("extra_assets/images/rstudio-logo.svg") }}"
```
