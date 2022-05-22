# Helm charts

Lost on the Internet, can't find a way to create a Helm v3 chart for your multi-pods, multi-YAML deployment ? Well, let's just say that once you see the light, it easy peasy !

As usual in my repos : this for practice and education. Don't try this at work, kids ! Once you know how to do this in your lab, make sure you secure this by every possible means.

I'll use the simple-microservice deployment that sits somewhere in my repos. The assumption is that all the YAML files in that repo can be deployed, and provide a working app. 

Start by creatinh an empty chart : 

```
helm create simple-micro
```

This creates an empty chart. If you have the linux `tree` command, look into it and check the structure. 

```
.
└── simple-micro
    ├── Chart.yaml
    ├── charts
    ├── templates
    │   ├── _helpers.tpl
    │   ├── micro-configmap.yaml
    │   ├── micro-deployment.yaml
    │   ├── micro-svc.yaml
    │   ├── redis-deployment.yaml
    │   ├── redis-ns.yaml
    │   ├── redis-svc.yaml
    │   └── tests
    └── values.yaml
```

To start easy, the important files and directories are : `Charts.yaml`, `values.yaml`, and the `templates/`. Remember, we have a working deployment, and we want to make our life easier, and deploy all files at once without too much hassle for once. 

- We will thus start by removing all files and subdirectories in the templates directory (except the file  `_helpers.tpl`). 
- Then, we'll edit the file `values.yaml`, delete all its content, and save it. 

- Next we'll copy all the YAML files from the simple-microservice repo under templates.

- Finally, we'll edit the file Charts.yaml to update the meta data. 

And that should be it. Run the command 

```
helm install micro-serv-by-helm simple-micro/
```

and you should be off to the races ! 

See ? Told you it was easy.

## Can we make this more complex ?

Yes we can ! 

What we did here is just bundled all our files together, and just turned them into a single entity to be deployed at once. 

But we can add a bit of flexibility, by removing hardcoded stuff from the original YAML file. 

So let's edit the files values.yaml. It's a YAML file, and a basic structure looks like this (I said "basic". We can make it a lot more complicated, but this isn't a YAML course ! ):

```
itemA:
  subItemA1: value1
  subItemA2: vaiue2

itemB:
  subItemB1: value3
  subItemB2: value4
```

These values can then be used in the YAML files you placed in the templates directory. And in these files, replace hardcoded values by their equivalent in the values.yaml file. 

For Helm's templating engine to work, the translation will be a dotted one : 

``` 
value1 = .Values.itemA.subItemA1
value2 = .Values.itemA.subItemA2
value3 = .Values.itemB.subItemB1
value4 = .Values.itemB.subItemB2
```

Let's give a clearer example maybe. Let's say you don't want to hardcode the value of the image used in the micro-service deployment, but rather make it a value that can easily be updated. In the file `values.yaml`, create the following : 

```
micro-serv:
  image: stratokumulus/micro-serviceapp
  tag: 1.0.4
```

and in the micro-deployment.yaml file in the template directory, in the pod specification, replace the hardcoded image with the values. 

From 
```
   spec: 
      containers:
        - name: micro-service
          image: stratokumulus/micro-serviceapp:18
```
to
```
   spec: 
      containers:
        - name: micro-service
          image: {{ .Values.micro.image }}:{{ .Values.micro.tag }} 
```

This makes changing images, version, ... a lot easier (especially if you reuse the variables at multiple places). 
