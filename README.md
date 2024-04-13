# JupyterHub Templates

The original upstream templates can be found here: https://github.com/jupyterhub/jupyterhub/tree/main/share/jupyterhub/templates

See: https://github.com/pangeo-data/pangeo-custom-jupyterhub-templates

https://discourse.jupyter.org/t/customizing-jupyterhub-on-kubernetes/1769/4

## Directory structure

The `templates` folder contains Jinja templates to be rendered by JupyterHub. This should be mounted in `/etc/jupyterhub/templates`.

The `static` folder contains CSS files and images used by the pages. This should be mounted in `/usr/local/share/jupyterhub/static/external`. Note that `/usr/local/share/jupyterhub/static` is already occupied and shouldn't be overwritten, and our mounted folder corresponds to `/hub/static/external/` in the URL, not `/hub/static`.

## Making Changes

1. Make your changes and get them on the master branch. This can be done by either committing straight to the branch or by making a new branch and merging.
1. Restart the jhub.

## Usage

Add to the Z2JH config file. Uses the `alpine/git` image as an initContainer to clone the repo and move the template files to the right places:
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
        - name: custom-templates-static
          mountPath: /extra-assets
  extraVolumes:
    - name: custom-templates
      emptyDir: {}
    - name: custom-templates-static
      emptyDir: {}
  extraVolumeMounts:
    - name: custom-templates
      mountPath: /usr/local/share/jupyter/hub/custom_templates
    - name: custom-templates-static
      mountPath: /usr/local/share/jupyter/hub/static/custom
  extraConfig:
    templates: |
      c.JupyterHub.template_paths = ['/usr/local/share/jupyter/hub/custom_templates/',
                                '/usr/local/share/jupyter/hub/templates/']
```
