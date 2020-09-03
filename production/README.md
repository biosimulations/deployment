# Biosimulations Deployment

This directory contains the files needed to deploy onto a cloud hosted Kubernetes cluster
In general, these files should work with any Kubernetes implementation, with the exception of some resources in [Ingress](#Ingress) and [Storage](#storage) that depend on provider specific resources.

## Apps

The apps folder contains the deployment and service specification for the various components such as the resource api and dispatch service.

## Resources

The resources folder contains the yaml files for shared resources, such as a Redis cluster, NATS cluster, and MongoDB cluster.

## Custom

The custom folder contains copies of the custom resource definitions used. These are kept current with the state of the cluster, and can be periodically updated

## Ingress

The ingress folder contains an ingress definition that is designed to work with Nginx-Ingress and Cert-Manager installed. It also contains the certificates to provision.

## Storage

Contains specific storage classes and claims that are used by the apps. Also contains the mechanisms to connecting to the backend storage providers

## Config

Contains sample files that handle the creation of config maps and secrets.
