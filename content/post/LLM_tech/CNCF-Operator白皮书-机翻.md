---
title: "CNCF Operator 白皮书 最终版（机翻）"
date: 2025-12-01
description: "CNCF Operator White Paper 最终版中英双语机翻"
tags: ["Kubernetes", "Operator", "云原生"]
categories: ["分布式系统"]
---

# CNCF Operator 白皮书 - 最终版 CNCF Operator White Paper - Final Version

目录 Table of Contents

=================



- [目录 Table of Contents](#table-of-contents)
- [概述 Executive Summary](#executive-summary)
- [简介 Introduction](#introduction)
- [本文档的目标 The Goal of this document](#the-goal-of-this-document)
- [目标读者 / 最低经验要求 Target Audience / Minimum Level of Experience](#target-audience--minimum-level-of-experience)
- [基础 Foundation](#foundation)
- [Operator 设计模式 Operator Design Pattern](#operator-design-pattern)
- [Operator 特征 Operator Characteristics](#operator-characteristics)
- [Kubernetes 中的 Operator 组件 Operator Components in Kubernetes](#operator-components-in-kubernetes)
- [Operator 能力 Operator Capabilities](#operator-capabilities)
- [安全 Security](#security)
- [Operator 开发者 Operator Developer](#operator-developer)
- [应用开发者（Operator-"用户"）Application Developer (Operator-"Users")](#application-developer-operator-users)
- [Kubernetes Operator 框架 Operator Frameworks for Kubernetes](#operator-frameworks-for-kubernetes) [CNCF Operator Framework](#cncf-operator-framework)
- [Kopf](#kopf)
- [kubebuilder](#kubebuilder)
- [Metacontroller - 轻量级 Kubernetes 控制器即服务 Metacontroller - Lightweight Kubernetes Controllers as a Service](#metacontroller---lightweight-kubernetes-controllers-as-a-service)
- [Operator 生命周期管理 Operator Lifecycle Management](#operator-lifecycle-management)
- [升级 Operator Upgrading the Operator](#upgrading-the-operator)
- [升级声明式状态 Upgrading the Declarative State](#upgrading-the-declarative-state)
- [管理 CRD 的关系 Managing Relations of CRDs](#managing-relations-of-crds)
- [Operator 用例 Use Cases for an Operator](#use-cases-for-an-operator) [Prometheus Operator](#prometheus-operator)
- [GitOps Operator Operator for GitOps](#operator-for-gitops)
- [成功模式 Successful Patterns](#successful-patterns)
- [管理单一类型的应用 Management of a single type of application](#management-of-a-single-type-of-application)
- [Operator 的 Operator Operator of Operators](#operator-of-operators)
- [每个控制器一个 CRD One CRD per Controller](#one-crd-per-controller)
- [在哪里发布和查找 Operator Where to publish and find Operators](#where-to-publish-and-find-operators)
- [延伸阅读 Further reading](#further-reading)
- [设计 Operator Designing Operators](#designing-operators)
- [需求分析 Requirement Analysis](#requirement-analysis)
- [自定义或第三方 Operator Custom or third-party Operator](#custom-or-third-party-operator)
- [选择正确的工具 Use the right Tool](#use-the-right-tool)
- [选择正确的编程语言 Use the right programming language](#use-the-right-programming-language)
- [根据需求设计 Operator Design your Operator according to your needs](#design-your-operator-the-according-to-your-needs)
- [参考文献 References](#references)
- [未来新兴模式 Emerging Patterns of the Future](#emerging-patterns-of-the-future)
- [Operator 生命周期管理 Operator Lifecycle Management](#operator-lifecycle-management-1)
- [策略感知 Operator Policy-Aware Operators](#policy-aware-operators)
- [参考文献 References](#references-1)
- [结论 Conclusion](#conclusion)
- [相关工作 Related Work](#related-work)
- [致谢 Acknowledgements](#acknowledgements)
- [贡献者 Contributors](#contributors)
- [审阅者 Reviewers](#reviewers)

## 1. 概述 Executive Summary

维护应用基础设施需要大量缺乏持久价值的重复性人工活动。·

Maintaining application infrastructure requires many repetitive human activities that are devoid of lasting value.·

·

计算机是执行精确任务、验证对象状态的首选方法，从而使得基础设施需求能够被编码化。Operator 提供了一种封装应用所需活动、检查和状态管理的方式。·

Computers are the preferred method of performing precise tasks, verifying the state of an object and therefore enabling the infrastructure requirements to be codified. An operator provides a way to encapsulate the required activities, checks and state management of an application.·

·

在 Kubernetes 中，Operator 通过扩展 API 功能提供智能化的动态管理能力。·

In Kubernetes, an operator provides intelligent, dynamic management capabilities by extending the functionality of the API.·

·

这些 Operator 组件使得常见流程的自动化以及能够持续适应环境的响应式应用成为可能。这反过来又使得开发更快速、错误更少、平均恢复时间更短，并且提高了工程团队的自主性。·

These operator components allow for the automation of common processes as well as reactive applications that can continually adapt to their environment. This in turn, allows for more rapid development with fewer errors, lower mean-time-to-recovery, and increased engineering autonomy.·

·

鉴于 Operator 模式日益流行，亟需一份参考文档来帮助新手和专家 alike 从社区认可的最佳实践中学习，以实现他们的目标。·

Given the rising popularity of the operator pattern, it has become incumbent for there to be a reference paper that helps both novice and expert alike to learn from the community endorsed best practices for achieving their goals.·

·

在本文档中，我们不仅概述了 Operator 的分类体系，还提出了 Operator 应用管理系统的推荐配置、实现和用例。·

In this document, we outline not only the taxonomy of an operator but the recommended configuration, implementation and use cases for an operator application management system.·

·

## 2. 简介 Introduction

本白皮书在比 Kubernetes 更广泛的语境下定义了 Operator。它描述了 Operator 的特征和组件，概述了当前常用的模式，并解释了它们与 Kubernetes 控制器的区别。·

This whitepaper defines operators in a wider context than Kubernetes. It describes their characteristics and components, gives an overview of common patterns currently in use and explains how they differ from Kubernetes controllers.·

·

此外，本文深入探讨了 Kubernetes 控制器的能力，包括备份、恢复和自动配置调优。还提供了对当前使用的框架、生命周期管理、安全风险和用例的进一步洞察。·

Additionally, it provides a deep dive into the capabilities of Kubernetes controllers, including backup, recovery and automatic configuration tuning. Further insights into frameworks currently in use, lifecycle management, security risks and use cases are provided.·

·

本文包含了可观测性、安全性和技术实现方面的最佳实践。·

This paper includes best practices including observability, security and technical implementation.·

·

最后以相关工作收尾，强调了它们在本白皮书之外能够带来的附加价值以及 Operator 的后续发展方向。·

It closes with related work, highlights the additional value they can bring beyond this whitepaper and the next steps for operators.·

·

### 2.1 本文档的目标 The Goal of this Document

本文档的目标是在 Kubernetes 和其他容器编排器的语境下，为云原生应用提供 Operator 的定义。·

The goal of this document is to provide a definition of operators for cloud native applications in the context of Kubernetes and other container orchestrators.·

·

### 2.2 目标读者 / 最低经验要求 Target Audience / Minimum Level of Experience

本文档面向应用开发者、Kubernetes 集群运维人员和服务提供商（内部或外部）——他们希望了解 Operator 及其能够解决的问题。它也可以帮助已经在研究 Operator 的团队了解何时何地使用它们最有效。本文假定读者具备基本的 Kubernetes 知识，例如熟悉 Pod 和 Deployment。·

This document is intended for application developers, Kubernetes cluster operators and service providers (internal or external) - who want to learn about operators and the problems they can solve. It can also help teams already looking at operators to learn when and where to use them to best effect. It presumes basic Kubernetes knowledge such as familiarity with Pods and Deployments.·

·

## 3. 基础 Foundation

Kubernetes 和其他编排器的成功在于它们对容器核心能力的关注。·

Kubernetes and the success of other orchestrators has been due to their focus on the main capabilities of containers.·

·

当企业开始云原生之旅时，处理更具体的用例（微服务、无状态应用）更有意义。·

While companies began their journey to cloud native, working with more specific use cases (microservices, stateless applications) made more sense.·

·

随着 Kubernetes 和其他容器编排器的声誉和可扩展性不断增长，需求也变得更加宏大。·

As Kubernetes and other container orchestrators grew their reputation and extensibility, requirements became more ambitious.·

·

使用编排器全生命周期功能的愿望也被转移到了高度分布式数据存储上。·

The desire to use the full lifecycle capabilities of an orchestrator was also transferred to highly distributed data stores.·

·

Kubernetes 原语默认并不是为管理状态而构建的。·

Kubernetes primitives were not built to manage state by default.·

·

仅依赖 Kubernetes 原语会给管理有状态应用需求带来困难，例如复制、故障转移自动化、备份/恢复和升级（_这些可能基于过于特定的事件而发生_）。·

Relying on Kubernetes primitives alone brings difficulty managing stateful application requirements such as replication, failover automation, backup/restore and upgrades (_which can occur based on events that are too specific_).·

·

Operator 模式可以用来解决状态管理的问题。·

The Operator Pattern can be used to solve the problem of managing state.·

·

通过利用 Kubernetes 内置能力（如自愈、协调）并将其与应用特定的复杂性相扩展，可以自动化任何应用的生命周期和运维，使其成为高能力的解决方案。·

By leveraging Kubernetes built-in capabilities such as self-healing, reconciliation and extending those along with application-specific complexities; it is possible to automate any application lifecycle, operations and turn it into a highly capable offering.·

·

人们通常认为 Operator 与 Kubernetes 是同义的。·

Operators are thought of as synonymous with Kubernetes.·

·

然而，将应用管理完全自动化的理念可以移植到其他平台。·

However, the idea of an application whose management is entirely automated can be exported to other platforms.·

·

本文的目的是将这一概念提升到比 Kubernetes 本身更高的层面。·

The aim of this paper is to bring this concept to a higher level than Kubernetes itself.·

·

### 3.1 Operator 设计模式 Operator Design Pattern

本节用高层概念描述该模式。·

This section describes the pattern with high-level concepts.·

·

下一节 _Kubernetes Operator 定义_ 将从 Kubernetes 对象和概念的角度描述该模式的实现。·

The next section _Kubernetes Operator Definition_ will describe the implementations of the pattern in terms of Kubernetes objects and concepts.·

·

Operator 设计模式定义了如何使用领域特定知识和声明式状态来管理应用和基础设施资源。该模式的目标是通过将领域特定知识编码到代码中并通过声明式 API 暴露出来，减少保持应用健康和良好维护状态所需的手动命令式工作（如何备份、扩缩容、升级...）。·

The operator design pattern defines how to manage application and infrastructure resources using domain-specific knowledge and declarative state. The goal of the pattern is to reduce the amount of manual imperative work (how to backup, scale, upgrade...) which is required to keep an application in a healthy and well-maintained state, by capturing that domain specific knowledge in code and exposing it using a declarative API.·

·

通过使用 Operator 模式，关于如何调整和维护资源的知识被编码到代码中，通常封装在单一服务（也称为控制器）内。·

By using the operator pattern, the knowledge on how to adjust and maintain a resource is captured in code and often within a single service (also called a controller).·

·

使用 Operator 设计模式时，用户只需描述应用和资源的期望状态。Operator 实现应该做出必要的变更使实际状态达到期望状态。Operator 还会持续监控实际状态并采取行动以保持其健康和一致（防止漂移）。·

When using an operator design pattern the user should only be required to describe the desired state of the application and resources. The operator implementation should make the necessary changes in the world so it will be in the desired state. The operator will also monitor the real state continuously and take actions to keep it healthy and in the same state (preventing drifts).·

·

Operator 的一般示意图中包含可以读取期望规约并能创建和管理所描述资源的软件。·

A general diagram of an operator will have software that can read the desired spec and can create and manage the resources that were described.·

·

![Operator 设计模式 Operator Design Pattern](img/02_1_operator_pattern.png)

Operator 模式由三个组件组成：·

The Operator pattern consists of three components:·

·

- 我们想要管理的应用或基础设施。 The application or infrastructure that we want to manage.·
- 一种领域特定语言，使用户能够以声明式方式指定应用的期望状态。 A domain specific language that enables the user to specify the desired state of the application in a declarative way.·
- 一个持续运行的控制器： A controller that runs continuously:·
- 读取并感知状态。 Reads and is aware of the state.·
- 当运维状态变更时自动执行动作。 Runs actions when operations state changes in an automated way.·
- 以声明式方式报告应用状态。 Report the state of the application in a declarative way.·

此设计模式将在后续章节中应用于 Kubernetes 及其 Operator。·

This design pattern will be applied on Kubernetes and its operators in the next sections.·

·

### 3.2 Operator 特征 Operator Characteristics

任何 Operator 的核心目的是用新的领域知识扩展其编排器底层 API。例如，Kubernetes 中的编排平台通过 Pod 和 Service 对象原生理解容器和四层负载均衡器之类的东西。Operator 为更复杂的系统和应用添加了新的能力。例如，prometheus-operator 引入了新的对象类型 _Prometheus_，扩展了 Kubernetes 对部署和运行 Prometheus 服务器的高层支持。·

The core purpose of any operator is to extend its orchestrator's underlying API with new domain knowledge. As an example, an orchestration platform within Kubernetes natively understands things like containers and layer 4 load balancers via the Pod and Service objects. An operator adds new capabilities for more complex systems and applications. For instance, a prometheus-operator introduces new object types _Prometheus_, extending Kubernetes with high-level support for deploying and running Prometheus servers.·

·

Operator 提供的能力可以归为三大类别：动态配置、运维自动化和领域知识。·

The capabilities provided by an operator can be sorted into three overarching categories: dynamic configuration, operational automation and domain knowledge.·

·

#### 3.2.1 动态配置 Dynamic Configuration

从软件开发的早期阶段开始，配置软件主要有两种方式：配置文件和环境变量。云原生世界创造了更新的流程，基于在启动时查询一个众所周知的 API。大多数现有软件依赖这两种选项的组合。Kubernetes 自然地提供了许多工具来实现自定义配置（如 ConfigMap 和 Secret）。由于大多数 Kubernetes 资源是通用的，它们不理解修改特定应用的任何细节。相比之下，Operator 可以定义新的自定义对象类型（自定义资源），以更好地在 Kubernetes 语境下表达特定应用的配置。·

Since the early stages of software development, there have been two main ways to configure software: configuration files and environment variables. The cloud native world created newer processes, which are based on querying a well-known API at startup. Most existing software relies on a combination of both of these options. Kubernetes naturally provides many tools to enable custom configuration (such as ConfigMaps and Secrets). Since most Kubernetes resources are generic, they don't understand any specifics for modifying a given application. In comparison, an operator can define new custom object types (custom resources) to better express the configuration of a particular application in a Kubernetes context.·

·

更好的验证和数据结构化可以减少小配置错误的可能性，并提高团队自助服务的能力。这消除了每个团队都需要掌握底层编排器或目标应用理解的传统要求。这可以包括渐进式默认值（用少量高层设置来填充由最佳实践驱动的配置文件），或自适应配置（如根据可用硬件或基于集群规模的预期负载来调整资源使用）。·

Allowing for better validation and data structuring reduces the likelihood of small configuration errors and improves the ability of teams to self-serve. This removes the requirement for every team to house the understanding of either the underlying orchestrator or the target application as would be traditionally required. This can include things like progressive defaults, where a few high-level settings are used to populate a best-practices-driven configuration file or adaptive configuration such as adjusting resource usage to match available hardware or expected load based on cluster size.·

·

#### 3.2.2 运维自动化 Operational Automation

除了自定义资源，大多数 Operator 还包含至少一个自定义控制器。这些控制器是像其他组件一样运行在编排器内部的守护进程，但它们连接到底层 API 并提供常见或重复性任务的自动化。这与编排器（如 Kubernetes）本身的实现方式相同。你可能已经在学习过程中见过 kube-controller-manager 或 cloud-controller-manager。然而，正如配置方面所展示的，Operator 可以用更高级的自动化来扩展和增强编排器，例如部署集群化软件、提供自动备份和恢复，或基于负载的动态扩缩容。·

Along with custom resources, most operators include at least one custom controller. These controllers are daemons that run inside the orchestrator like any other but connect to the underlying API and provide automation of common or repetitive tasks. This is the same way that orchestrators (like Kubernetes) are implemented. You may have seen kube-controller-manager or cloud-controller-manager mentioned in your journey so far. However, as was demonstrated with configuration, operators can extend and enhance orchestrators with higher-level automation such as deploying clustered software, providing automated backups and restores, or dynamic scaling based on load.·

·

将这些常见运维任务编码到代码中，可以确保它们以标准化的方式可重复、可测试和可升级。让人类远离频繁任务的循环还能确保步骤不会被遗漏或省略，任务的不同部分不会彼此失去同步。如前所述，这通过减少在无聊但重要的维护任务（如应用备份）上花费的时间，提高了团队自主性。·

By putting these common operational tasks into code, it can be ensured they will be repeatable, testable and upgradable in a standardized fashion. Keeping humans out of the loop on frequent tasks also ensures that steps won't be missed or excluded and that different pieces of the task can't drift out of sync with each other. As before, this allows for improved team autonomy by reducing the hours spent on boring-but-important upkeep tasks like application backups.·

·

#### 3.2.3 领域知识 Domain Knowledge

与运维自动化类似，可以将特定软件或流程的专业领域知识编码到 Operator 中。一个常见的例子是应用升级。简单的无状态应用可能只需要 Deployment 的滚动升级；而数据库和其他有状态应用通常需要非常特定的有序步骤来安全地执行升级。Operator 可以自主处理这些，因为它知道当前版本和请求的版本，并能在需要时运行专门的升级代码。更一般地说，这适用于前云原生环境中使用手动检查清单的任何场景（实际上是将 Operator 用作可执行的运行手册）。·

Similar to operational automation, it can be written into an operator to encode specialized domain knowledge about particular software or processes. A common example of this is application upgrades. While a simple stateless application might need nothing more than a Deployment's rolling upgrade; databases and other stateful applications often require very specific steps in sequence to safely perform upgrades. The operator can handle this autonomously as it knows your current and requested versions and can run specialized upgrade code when needed. More generally, this can apply to anything a pre-cloud-native environment would use manual checklists for (effectively using the operator as an executable runbook).·

·

另一种利用自动化领域知识的常见方式是错误修复。例如，Kubernetes 内置的修复行为基本上始于并止于"重启容器直到它工作"，这是一个强大的解决方案，但通常不是最好或最快的方案。·

Another common way to take advantage of automated domain knowledge is error remediation. For example, the Kubernetes built-in remediation behaviors mostly start and end with "restart container until it works" which is a powerful solution but often not the best or fastest solution.·

·

Operator 可以监控其应用并针对错误做出特定行为来解决问题，如果无法自动解决则升级该问题。这可以降低 MTTR（平均恢复时间），也可以减少反复出现的问题导致的运维疲劳。·

An operator can monitor its application and react to errors with specific behavior to resolve the error or escalate the issue if it can't be automatically resolved. This can reduce MTTR (mean time to recovery) and also reduce operator fatigue from recurring issues.·

·

### 3.3 Kubernetes 中的 Operator 组件 Operator Components in Kubernetes

*"Operator 是一个理解两个领域的 Kubernetes 控制器：Kubernetes 和其他某个领域。通过结合两个领域的知识，它可以自动化通常需要理解两个领域的人类运维来完成的任务"*·

(Jimmy Zelinskie, https://github.com/kubeflow/tf-operator/issues/300#issuecomment-357527937)

·

*An operator is a Kubernetes controller that understands 2 domains: Kubernetes and something else. By combining knowledge of both domains, it can automate tasks that usually require a human operator that understands both domains"*·
(Jimmy Zelinskie, https://github.com/kubeflow/tf-operator/issues/300#issuecomment-357527937)

![Operator 全景图 Operator Big Picture](img/02_2_operator.png)

Operator 使得运维知识能够扩展 Kubernetes API。·

Operators enable the extension of the Kubernetes API with operational knowledge.·

·

这是通过组合 Kubernetes 控制器和描述期望状态的被监控对象来实现的。控制器可以监控一个或多个对象，这些对象可以是 Kubernetes 原语（如 Deployment、Service），也可以是集群外部的资源（如虚拟机或数据库）。·

This is achieved by combining Kubernetes controllers and watched objects that describe the desired state. The controller can watch one or more objects and the objects can be either Kubernetes primitives such as Deployments, Services or things that reside outside of the cluster such as Virtual Machines or Databases.·

·

期望状态在此指的是在代码中定义的、Operator 被配置来管理的任何资源。相应地，当前状态指的是这些资源的已部署实例。·

The desired state refers hereby to any resource that is defined in code and which the operator is configured to manage. Subsequently, the current state references the deployed instance of those resources.·

·

控制器将使用协调循环不断比较期望状态和当前状态，确保被监控的对象以定义的方式过渡到期望状态。·

The controller will constantly compare the desired state with the current state using the reconciliation loop which ensures that the watched objects get transitioned to the desired state in a defined way.·

·

期望状态封装在一个或多个 Kubernetes 自定义资源中，控制器包含将对象（如 Deployment、Service）带到其目标状态所需的运维知识。·

The desired state is encapsulated in one or more Kubernetes custom resources and the controller contains the operational knowledge which is needed to get the objects (such as deployments, services) to their target state.·

·

#### 3.3.1 Kubernetes 控制器 Kubernetes Controllers

Kubernetes 控制器负责日常任务，确保特定资源类型表达的期望状态与当前状态匹配。例如，Deployment 控制器确保所需数量的 Pod 副本在运行，当一个 Pod 被删除或失败时会启动新 Pod。·

A Kubernetes Controller takes care of routine tasks to ensure the desired state expressed by a particular resource type matches the current state. For instance, the Deployment controller takes care that the desired amount of pod replicas is running and a new pod spins up, when one pod is deleted or fails.·

·

从技术上讲，典型的控制器和 Operator 之间没有区别。通常所说的区别在于 Operator 包含的运维知识。因此，控制器是实现，Operator 是使用自定义控制器和 CRD 的模式，自动化是其所追求的目标。因此，当创建自定义资源时启动一个 Pod、之后 Pod 被销毁的控制器可以描述为简单控制器。如果控制器具有如何升级或从错误中修复的运维知识，那它就是一个 Operator。·

Technically, there is no difference between a typical controller and an operator. Often the difference referred to is the operational knowledge that is included in the operator. Therefore, a controller is the implementation, and the operator is the pattern of using custom controllers with CRDs and automation is what is looking to be achieved with this. As a result, a controller which spins up a pod when a custom resource is created, and the pod gets destroyed afterwards can be described as a simple controller. If the controller has operational knowledge like how to upgrade or remediate from errors, it is an operator.·

·

#### 3.3.2 自定义资源和自定义资源定义 Custom Resources and Custom Resource Definitions

自定义资源用于在 Kubernetes 中存储和检索结构化数据，作为默认 Kubernetes API 的扩展。·

Custom resources are used to store and retrieve structured data in Kubernetes as an extension of the default Kubernetes API.·

·

在 Operator 的场景下，自定义资源包含资源（如应用）的期望状态，但不包含实现逻辑。这些信息可以是应用组件的版本信息，也可以是应用的已启用功能，或应用的备份信息等。自定义资源定义（CRD）定义了此类对象的外观，例如存在哪些字段以及 CRD 的命名方式。这样的 CRD 可以使用工具（如 Operator SDK）脚手架生成，也可以手动编写。·

In the case of an operator, a custom resource contains the desired state of the resource (e.g. application) but does not contain the implementation logic. Such information could be the version information of application components, but also enabled features of an application or information where backups of the application could be part of this. A custom resource definition (CRD) defines how such an object looks like, for example, which fields exist and how the CRD is named. Such a CRD can be scaffolded using tools (as the operator SDK) or be written by hand.·

·

以下示例说明了这样一个自定义资源实例定义可能的样子：·

The following example illustrates, how such an custom resource instance definition could look like:·



```yaml
apiVersion: example-app.appdelivery.cncf.io/v1alpha1
kind: ExampleApp
metadata:
  name: appdelivery-example-app
spec:
  appVersion: 0.0.1
  features:
    exampleFeature1: true
    exampleFeature2: false
  backup:
    enabled: true
    storageType: "s3"
    host: "my-backup.example.com"
    bucketName: "example-backup"
  status:
    currentVersion: 0.0.1
    url: https://myloadbalancer/exampleapp/
    authSecretName: appdelivery-example-app-auth
    backup:
      lastBackupTime: 12:00
```

此示例表示一个名为 "appdelivery-example-app"、类型为 "ExampleApp" 的自定义资源。·

This example represents a custom resource with the name "appdelivery-example-app" of the kind "ExampleApp".·

·

"spec" 部分是用户声明期望状态的地方。此示例声明应部署 appVersion 0.0.1，启用一个功能、禁用另一个功能。此外，应对此应用进行备份，并使用 S3 存储桶。·

The "spec" section is where the user can declare the desired state. This example declares that appVersion 0.0.1 should be deployed with one feature enabled and another disabled. Furthermore, backups of this application should be made, and a s3 bucket should be used.·

·

"status" 部分是 Operator 向用户传达有用信息的地方。在此示例中，状态显示了当前部署的版本。如果它与 spec 中的 "appVersion" 不同，则用户可以预期 Operator 正在努力部署 spec 中请求的版本。状态部分中的其他常见信息包括如何连接到应用以及应用的健康状况。·

The "status" section is where the operator can communicate useful information back to the user. In this example, the status shows the current deployed version. If it is different from the "appVersion" in the spec, then the user can expect that the operator is working to deploy the version requested in the spec. Other common information in the status section includes how to connect to an application and the health of the application.·

·

#### 3.3.3 控制循环 Control Loop

Kubernetes 控制器中的控制（协调）循环确保用户使用 CRD 声明的状态与应用的状态匹配，同时确保状态之间的过渡按预期工作。一个常见的用例是在升级应用时迁移数据库模式。控制循环可以在特定事件（如 CRD 的变更）上触发，也可以基于时间触发，如在定义的时间备份数据。·

The control (reconciliation) loop in a Kubernetes controller ensures that the state that the user declares using a CRD matches the state of the application, but also that the transition between the states works as intended. One common use-case could be the migration of database schemes when upgrading an application. The control loop can be triggered on specific events, as a change on the crd, but also time-based, like for backing up data at a defined time.·

·

### 3.4 Operator 能力 Operator Capabilities

Operator 能够通过解决许多不同的任务来协助运维应用或其他托管组件。谈到 Operator 时，第一个也是最知名的能力是安装和升级有状态应用。然而，Operator 可以在安装/升级时不需要人工干预的情况下管理应用的完整生命周期。·

An operator is able to assist with operating an application or other managed components by solving many different tasks. When talking about operators, the first and most well known capability is the ability of installing and upgrading stateful applications. However, an operator could manage the full lifecycle of an application without requiring manual input on installation/upgrades.·

·

以下各节应概述 Operator 可能具有的能力，以及如果 Operator 实现了这些能力，用户可以期望什么。·

The following sections should give an overview about capabilities an operator could have and what a user can expect if an operator implements these capabilities.·

·

#### 3.4.1 安装应用 / 接管应用 Install an Application / Take Ownership of an Application

Operator 应该能够配置和设置所有必需的资源，使安装过程中不需要人工操作。Operator 必须检查和验证已配置的资源是否按预期工作并可以使用。·

An operator should be able to provision and set up all the required resources, so no manual work would be required during the installation. An operator must check and verify that resources that were provisioned are working as expected, and ready to be used.·

·

Operator 还应该能够识别在安装过程之前已配置的资源，仅接管它们以供后续使用。在这种情况下，接管过程应该是无缝的，不会导致停机。接管过程的目的是使资源能够轻松迁移到 Operator 管理。·

An operator should also be able to recognize resources that were provisioned before the installation process, and only take ownership of them for later use. In this case, the ownership process should be seamless and not cause downtime. The ownership process purpose is to enable easy migration of resources to the operator.·

·

Operator 应该在此过程中报告资源的版本及其健康状态。·

An Operator should report the version of the resources and their health status during the process.·

·

#### 3.4.2 升级应用 Upgrade an Application

Operator 应该能够升级应用/资源的版本。Operator 应该知道如何更新所需的依赖项并执行自定义命令（如运行数据库迁移）。·

An operator should be able to upgrade the version of the application/resources. The operator should know how to update the required dependencies and execute custom commands such as running a database migration.·

·

Operator 应该监控更新过程，如果出现问题则回滚。·

An operator should monitor the update and rollback if there was a problem during the process.·

·

Operator 应该在此过程中报告资源的版本及其健康状态。如果出现错误，报告的版本应该是当前正在使用的版本。·

An operator should report the version of the resources and their health status during the process. If there was an error, the version reported should be the version that is currently being used.·

·

#### 3.4.3 备份 Backup

此能力适用于管理数据的 Operator，确保 Operator 能够创建一致的备份。备份应以这样的方式进行：Operator 的用户可以确信，如果数据丢失或受损，可以恢复到之前的版本。此外，提供的状态信息应提供关于备份上次运行时间和位置的洞察。·

This capability is for operators that manage data and ensure that the operator is able to create consistent backups. This backup should be done in a way that the user of the operator can be certain that the previous version can be restored if data is lost or compromised. Furthermore, the status information provided should give insights about when the backup last ran and where it is located.·

·

![示例备份流程 Example Backup Process](plantuml/backup-sequence.png)

上图展示了这样的流程可能的样子。首先，备份由人工或其他触发器（如时间触发器）触发。Operator 指示其监控的资源（应用）设置一致的状态（如一致性快照）。之后，应用的数使用适当的工具备份到外部存储。这可以是单步过程（直接备份到外部存储），也可以是多步过程，如先写入持久卷再写入外部存储。外部存储可以是本地的 NFS/CIFS 共享（或任何其他网络文件系统），也可以是云提供商基础设施上的对象存储/存储桶。无论备份失败还是成功，备份的状态（包括备份的应用版本和备份位置）可能会写入自定义资源的状态部分。·

The above illustration shows how such a process could look like. At first, the backup gets triggered either by a human or another trigger (e.g. time-trigger). The operator instructs its watched resource (application) to set up a consistent state (like a consistent snapshot). Afterwards, the data of the application gets backed up to external storage using appropriate tools. This could either be a one-step process (backup directly to external storage) or in multiple steps, like writing to a persistent volume at first and to the external storage afterwards. The external storage might be an NFS/CIFS share (or any other network file system) on-premises, but also an object store/bucket on a cloud provider infrastructure. Whether the backup failed or succeeded, the state (of the backup) including the backed-up application version and the location of the backup might be written to the status section of the custom resource.·

·

#### 3.4.4 从备份恢复 Recovery from backup

Operator 的恢复能力可以协助用户从成功的备份中恢复应用状态。因此，应用状态（应用版本和数据）应被恢复。·

The recovery capability of an operator might assist a user in restoring the application state from a successful backup. Therefore, the application state (application version and data) should be restored.·

·

实现方式可能有很多种。一种可能的方式是当前应用状态也被备份（包括配置），所以用户只需为应用创建一个自定义资源并指向备份即可。Operator 会读取配置，恢复应用版本并恢复数据。另一种可能的方案是用户只备份了数据，可能需要指定使用的应用版本。无论哪种方式，Operator 都确保在使用指定备份的数据后应用能够正常运行。·

There might be many ways to achieve this. One possible way could be that the current application state also gets backed up (including configuration), so the user only has to create a custom resource for the application and point to the backup. The operator would read the configuration, restore the application version and restore the data. Another possible solution might be that the user only backed up the data and might have to specify the application version used. Nevertheless, in both ways, the operator ensures that the application is up and running after using the data from the backup specified.·

·

#### 3.4.5 自动修复 Auto-Remediation

Operator 的自动修复能力应确保它能够从更复杂的失败状态中恢复应用，这些失败状态可能无法被健康检查（存活和就绪探针）等机制处理或检测到。因此，Operator 需要对应用有深入的理解。这可以通过可能指示应用故障或错误的指标来实现，也可以通过处理 Kubernetes 健康检查等机制来实现。·

The auto-remediation capability of an operator should ensure that it is able to restore the application from a more complex failed state, which might not be handled or detected by mechanisms such as health checks (live and readiness probes). Therefore, the operator needs to have a deep understanding of the application. This can be achieved by metrics that might indicate application failures or errors, but also by dealing with kubernetes mechanisms like health checks.·

·

一些示例可能是：·

Some examples might be:·

·

- 如果在版本变更后一定数量的 Pod 启动不成功，则回滚到上次已知配置。在某些情况下，重启应用可能是一个短期解决方案，这也可以由 Operator 完成。 Rolling back to the last known configuration if a defined amount of pod starts is unsuccessful after a version change. In some points a restart of the application might be a short-term solution which also could be done by the operator.·
- 也可以想象 Operator 通知依赖服务的另一个 Operator 后端系统当前不可达（以采取修复措施）。 It could also be imaginable that an operator informs another operator of a dependent service that a backend system is not reachable at the moment (to take remediation actions).·

在任何情况下，此能力使 Operator 能够采取行动保持系统正常运行。·

In any situation, this capability enables the operator to take actions to keep the system up and running.·

·

#### 3.4.6 监控/指标 - 可观测性 Monitoring/Metrics - Observability

虽然被管理的应用应该为自身提供遥测数据，但 Operator 可以提供关于自身行为的指标，并仅提供应用状态的高层概览（如同自动修复所可能的）。此外，Operator 提供的典型遥测数据可以是修复操作的次数、备份的持续时间，也可以是关于最后处理的错误或运维任务的信息。·

While the managed application should provide the telemetry data for itself, the operator could provide metrics about its own behavior and only provide a high level overview about the applications state (as it would be possible for auto-remediation). Furthermore, typical telemetry data provided by the operator could be the count of remediation actions, duration of backups, but also information about the last errors or operational tasks which were handled.·

·

#### 3.4.7 扩缩容 Scaling

扩缩容是 Operator 可以管理的 Day-2 运维的一部分，以保持应用/资源正常运行。扩缩容能力不要求扩缩容是自动化的，只要求 Operator 知道如何进行水平和垂直扩缩容。·

Scaling is part of the day-2 operations that an operator can manage in order to keep the application / resources functional. The scaling capability doesn't require the scaling to be automated, but only that the operator will know how to change the resources in terms of horizontal and vertical scaling.·

·

Operator 应该能够增加或减少其拥有的任何资源，如 CPU、内存、磁盘大小和实例数量。·

An operator should be able to increase or decrease any resource that it owns, such as CPU, memory, disk size and number of instances.·

·

理想情况下，扩缩容操作不会导致停机。扩缩容操作在所有资源处于一致状态并可以使用时结束，因此 Operator 应该验证所有资源的状态并报告。·

Ideally the scaling action will be without downtime. Scaling action ends when all the resources are in consistent state and ready to be used, so an operator should verify the state of all the resources and report it.·

·

#### 3.4.8 自动扩缩容 Auto-Scaling

Operator 应该能够基于持续收集的指标和阈值来执行扩缩容能力。Operator 应该能够自动增加和减少其拥有的每个资源。·

An operator should be able to perform the scaling capability based on metrics that it collects constantly and according to thresholds. An operator should be able to automatically increase and decrease every resource that it's own.·

·

Operator 应该遵守最小和最大值的基本扩缩容配置。·

An operator should respect basic scaling configuration of min and max.·

·

#### 3.4.9 自动配置调优 Auto-Configuration tuning

此能力应使 Operator 能够管理被管理应用的配置。例如，Operator 可以根据运维环境（如 Kubernetes）或 DNS 名称的变更来调整应用的内存设置。此外，Operator 应该能够以无缝的方式处理配置变更，例如如果配置变更需要重启，则应触发重启。·

This capability should empower the operator to manage the configuration of the managed application. As an example, the operator could adopt memory settings of an application according to the operation environment (e.g. Kubernetes) or the change of DNS names. Furthermore, the operator should be able to handle configuration changes in a seamless way, e.g. if a configuration change requires a restart, this should be triggered.·

·

这些能力对用户应该是透明的。用户应该有可能在愿意时覆盖此类自动配置机制。此外，自动重新配置应有良好的文档记录，以便用户能够理解基础设施上正在发生什么。·

These capabilities should be transparent to the users. The user should have the possibility to override such auto-configuration mechanisms if they want to do so. Furthermore, automatic reconfigurations should be well-documented in a way that the user could comprehend what is happening on the infrastructure.·

·

#### 3.4.10 卸载 / 断开 Uninstalling / Disconnect

当删除声明式请求的状态（大多数情况下是自定义资源）时，Operator 应该允许两种行为：·

When deleting the declarative requested state (in most cases a custom resource), an operator should allow two behaviors:·

·

- 卸载：Operator 应该能够完全移除或删除每个被管理的资源。 Uninstalling: An operator should be able to completely remove or delete every managed resource.·
- 断开：Operator 应该停止管理已配置的资源。 Disconnecting: An operator should stop managing the provisioned resources.·

两个过程都应应用于 Operator 直接配置的每个资源。·

Both processes should be applied to every resource that the operator directly provisioned.·

·

Operator 应该以声明式方式报告过程中的任何失败（例如使用[状态字段](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#object-spec-and-status)）。

An operator should report any failure in the process in a declarative way (using the [status field](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#object-spec-and-status) for example).·

·

## 4. 安全 Security

![Operator 模型 Operator Model](img/04_1_operator_model.png)

Operator 旨在通过 Kubernetes API 服务器使用自定义资源定义来管理其状态和配置。它们管理的下级 API 资源（通常是运行有状态应用的 Pod）也通过 Kubernetes API 管理其生命周期和支持性的 RBAC、Service 等。在某些情况下，Operator 还会通过网络与应用的 API 交互。所有这些路由都提供了损害 Operator 及其资源的可能性，应按照下面列出的最佳实践进行保护。·

Operators are intended to manage their state and configuration via the Kubernetes API server using the Custom Resource Definition. The subordinate API resources they manage (often pods running stateful applications) also have their lifecycle and supporting RBAC, services, etc. managed via the Kubernetes API. In some cases, the operator will also interact with the application's API across the network. All of these routes offer the potential to compromise the operator and its resources and should be protected in line with best practices laid out below.·

·

### 4.1 Operator 开发者 Operator Developer

Operator 开发者应该了解 Operator 引入的安全风险并记录其安全使用方式。在开发 Operator 时，重要的是关注关键领域，如透明度和文档、Operator 范围和漏洞分析。·

Operator developers should be aware of the security risks an operator introduces and document its secure use. While developing an operator it's important to focus on key areas such as transparency and documentation, operator scope, and vulnerability analysis.·

·

#### 4.1.1 透明度和文档 Transparency and Documentation

在开发 Operator 的过程中，开发者应该清楚了解它在 Kubernetes 中如何工作和交互。当开发者从开发转向发布 Operator 时，应该向用户提供对 Operator 做什么以及如何做的清晰理解。·

During the development of an operator, a developer should have a clear understanding of how it will work and interface within Kubernetes. As developers shift from development to publishing the operator, users should be provided with a clear understanding of what the operator does, and how.·

·

你写了一些引以为傲的东西，但从最终用户的角度想一想：他们是否应该信任来自互联网的源代码，让一个 Operator 在可能庞大且昂贵的集群上以管理员权限运行，或者处理敏感信息？开发者能做的任何帮助用户快速上手的事情——软件如何工作、如何安全保障、对集群可能有什么影响——都会让用户更容易采用该软件。·

You've written something you're proud of, but think of this from the end user's point of view: Should they trust source code from the internet, an operator to run with administrative access on their cluster which may be large and costly, or maybe handling sensitive information? Anything the developer can do to help a user get up to speed with their software, how it works, how it's secured, and what effects it might have on their cluster will make it easier for them to adopt the software.·

·

以下是一些可以帮助用户做出是否使用 Operator 的明智决定的条目：·

Here are some items that can help users make informed decisions about if they should use an operator:·

·

- Operator 的通信方式和对象的描述性图表（威胁模型）是帮助用户了解如何安全配置和为 Operator 应用策略的良好起点。 Descriptive diagram (threat model) of how the operator is communicating and with what is a good start to helping a user understand how they must secure it and apply policy for the operator.·
- 软件的预期使用用例，以保持在合规范围内，否则可能面临超出范围的漏洞风险。 Use case of how the software is intended to be used in order to stay in scope for compliance or you risk vulnerability outside that scope.·
- 记录的 RBAC 范围、威胁模型、通信端口、可用的 API 调用、Pod 安全策略要求（或其他策略引擎要求），或为 Kubernetes 开发的任何其他策略引擎要求（如 OPA）。 Documented RBAC scopes, threat model, communication ports, API calls available, pod security policy requisites (or other policy engine requisites), or any other policy engine requisites developed for Kubernetes such as OPA.·
- 安全报告、披露和事件响应流程：如果有人发现潜在的安全问题，应该联系谁，可以期望什么类型的响应？ Security reporting, disclosure, and incident response processes: If someone finds a potential security issue, who should they contact and what type of response should they expect?·
- 通过暴露的端点、日志级别或日志聚合来连接日志和监控。 Logging and monitoring attachment through exposed endpoints, log levels, or log aggregation.·
- Operator 问题、功能和版本跟踪。 Operator issue, feature, version tracking.·
- 如果项目过去有过安全披露，在网页上列出这些披露（及其 CVE ID）是建立用户信任的有力步骤。每个人在某个时候都会遇到安全问题——如何处理它们显示了一个项目的成熟度。 If the project has had security disclosures in the past, listing these disclosures (and their CVE IDs) on a web page is a strong step in building trust with users. Everyone will have security issues at some point - how they are handled displays the maturity of a project.·

关于开发过程安全性的更多想法，读者可能希望参阅 CNCF 安全 TAG 的[自我评估问卷](https://github.com/cncf/tag-security/blob/main/community/assessments/guide/self-assessment.md)。

For further ideas around the security of the development process, the reader may wish to review the CNCF Security TAG's [self-assessment questionnaire](https://github.com/cncf/tag-security/blob/main/community/assessments/guide/self-assessment.md).

·

#### 4.1.2 Operator 范围 Operator Scope

Operator 有许多用例，你可以为其设计的范围几乎没有限制。为了明确 Operator 的安全性质，每个范围都应有清晰的通信。可以使用的一般范围包括集群范围 Operator、命名空间 Operator 和外部 Operator。为了最好地保护它们，需要了解通信、创建的任何 API、控制器及其职责，以及任何应用指标端点。如果在 Operator 中提供了这些信息，就可以用来在实现范围内进一步保护 Operator 应用。如果不提供这些信息，可能会面临各种攻击的风险。·

There are many use cases for operators and there is virtually no limit in the scope of what you can design it for. In order to be clear about the secure nature of an operator there should be clear communication involved with each scope. The general scope's which could be used are cluster-wide operators, namespace operators, and external operators. In order to best secure them, there needs to be an understanding of the communication, any API's created, controllers and their responsibility, and any application metric endpoints. If this information is provided with the operator it can be used to further secure the operator application within the scope of implementation. If the information is not provided you can be left vulnerable to a myriad of attacks.·

·

**集群范围 Operator** 的目的是在整个集群中执行自定义资源，无论这些资源是否位于其他命名空间中。·

**Cluster-wide Operators** exist to execute custom resources across a cluster no matter if those resources are living in another namespace or not.·

·

**命名空间 Operator** 的目的是在命名空间内执行自定义资源。通常会应用策略引擎策略将范围限制在命名空间内，仅与命名空间内的 Pod 通信。这本质上被认为更安全，但同样的规则仍然适用。·

**Namespace Operators** exist to execute custom resources within a namespace. Usually there are policy engine policies applied to jail the scope within the namespace and only communicate with pods within the namespace. This is considered more secure by nature, but the same rules apply.·

·

**外部 Operator** 的目的是执行集群外部的自定义资源。同样的规则适用，此外为了保护此范围，我们必须了解从集群到外部组件的通信性质。·

**External Operators** exist to execute custom resources that are external to the cluster. The same rules apply, in addition to secure this scope we must know the nature of the communication from the cluster to the external component.·

·

虽然本文也从用户角度讨论范围，但 Operator 的设计方式将在很大程度上影响生产环境中可对其应用的安全控制类型。通常从宽松的权限开始，并打算在发布前应用安全概念；花一些时间在开发者开始工作时思考 Operator 的安全设计，将使这个过程对开发者和他们的用户更容易。·

While this paper also discusses scoping from a user point-of-view, how an operator is designed will weigh heavily on the type of security controls which can be applied against it in production. It is common to start with lax permissions, and intentions to apply security concepts before release; Spending some time thinking about the security design of the operator as developers begin work on it will make this process much easier for developers and their users.·

·

#### 4.1.3 漏洞分析 Vulnerability Analysis

专注于 Operator 的开发和安全性，Operator 开发者必须采取一些步骤来确保进行了验证和适当的安全分析。遵循 CNCF 云原生安全白皮书中的指南，有一个清晰的[关注层](https://github.com/cncf/tag-security/blob/main/community/resources/security-whitepaper/v2/cloud-native-security-whitepaper.md#cloud-native-layers)生命周期流程来定义 Operator 开发者的关注层。所有三个层都应遵守，在 Operator 开发者范围内严格关注开发和分发层。开发和分发层中有许多详细的指南，有助于对供应链进行健全的漏洞分析，确保正在开发的 Operator 经过签名和信任以获得最佳完整性。CNCF [云原生安全白皮书](https://github.com/cncf/tag-security/blob/main/community/resources/security-whitepaper/v2/cloud-native-security-whitepaper.md)可通过此链接获取。

Being focused on the development and security of the operator, there are steps that must be taken as an operator developer to ensure validation and proper security analysis has been done. Following the guidelines in the CNCF Cloud Native Security Whitepaper there is a clear lifecycle process which defines the [layers of concern](https://github.com/cncf/tag-security/blob/main/community/resources/security-whitepaper/v2/cloud-native-security-whitepaper.md#cloud-native-layers) for the operator developer. All three layers should be adhered to with a strict focus on the develop and distribute layers in the scope of the operator developer. There are many detailed guidelines in the development and distribution layers that will help to apply sound vulnerability analysis to supply chain to ensure that the operator being developed is signed and trusted for the best integrity. The CNCF [Cloud Native Security Whitepaper](https://github.com/cncf/tag-security/blob/main/community/resources/security-whitepaper/v2/cloud-native-security-whitepaper.md) is available at this link.·

·

除了供应链之外，还需要专注于对 Operator 进行威胁建模，以使开发者保持约束，并确保没有遗漏可能为攻击敞开大门的内容。检查威胁的基础模型可在 CNCF 云原生安全白皮书的[威胁建模](https://github.com/cncf/tag-security/blob/main/community/resources/security-whitepaper/v2/cloud-native-security-whitepaper.md#threat-modeling)部分中找到。

In addition to the supply chain there needs to be a focus on performing a threat model of the operator to keep the developer in check and also make sure that there was nothing incidentally missed that could leave the door open for attack. The foundational model for checking for threats can be observed in the CNCF Cloud Native Security Whitepaper on [Threat Modeling](https://github.com/cncf/tag-security/blob/main/community/resources/security-whitepaper/v2/cloud-native-security-whitepaper.md#threat-modeling).

·

### 4.2 应用开发者（Operator-"用户"）Application Developer (Operator-"Users")

Operator 代表用户执行管理任务，如卷创建/挂载、应用部署和证书管理。由于用户将控制权委托给 Operator，必须提供机器授权以执行所需的操作，但也必须注意不要授予超出 Operator 执行角色所需权限的更多权限。·

Operators perform administrative tasks on the user's behalf such as volume creation/attachment, application deployment, and certificate management. As the user is delegating control to the operator, it is essential to provide machine authorization to perform the actions needed, but one must also be careful to not grant more privileges than necessary for the operator to perform its role.·

·

部署 Operator 会授予第三方软件对 Kubernetes 命名空间或集群的某种级别的访问权限。虽然使用 Operator 不需要安全专业知识，但以下 Kubernetes 概念突出了使用 Operator 时的安全准备：·

Deployment of an operator grants third-party software some level of access to a Kubernetes namespace or cluster. While security expertise is not required to use operators, the following Kubernetes concepts highlight security preparation when using an operator:·

·

**命名空间（Namespaces）** 是分组和隔离一组资源的主要方式之一。关于 Operator，用户应该考虑 Operator 需要与哪些命名空间工作。虽然可能存在某个 Operator 需要访问整个集群的用例，但 2021 年的常见用例是 Operator 在 Kubernetes 中与特定应用工作，因此通常为该应用和相关资源/Operator 提供一个命名空间是有意义的。为了进一步减少 Operator 与下级资源命名空间中任何松散或被盗 RBAC 的关联，为 Operator 提供专用命名空间可提供更多隔离。·

**Namespaces** are one of the primary ways of grouping and cordoning a group of resources. In regards to an operator, the user should consider what namespaces the operator needs to work with. While there may be some use cases where a single operator needs access to the whole cluster, it seems the common use case in 2021 is for an operator to work with a specific application within Kubernetes, so it usually makes sense to provide a namespace for that application and related resources/operators. To further reduce the operator's separation from any loose or stolen RBAC in the subordinate resource's namespace, a dedicated namespace for the operator provides more separation.·

·

**基于角色的访问控制（RBAC）** 在 Kubernetes 的现代版本中可用。在授予 Operator 资源访问权限时，重点应该是授予 Operator 执行任务所需的最有限的权限集。这意味着只在绝对必要时才授予 ClusterRole，而是为特定资源/命名空间授予特定权限。用户指南中的[使用 RBAC 授权](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)章节详细介绍了此主题。Operator 构建工具包（如 Operator SDK）使用开发者可能未为其特定 Operator 优化的通用 RBAC 默认值。服务账户身份在集群外部的权限包括在其他 Kubernetes 集群中拥有权限的联邦和跨集群 Operator。随着 Operator 越来越多地用于管理集群外和云资源，应配置云 IAM 集成权限以防止云账户被受损的 Operator 接管。·

**Role-Based Access Controls** are available in modern releases of Kubernetes. When granting an operator access to resources, the focus should be on granting the most limited set of permissions needed for the operator to perform its task. This means only grant ClusterRoles if absolutely necessary, but granting specific permissions for specific resources/namespaces. The [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) chapter of the user guide covers this topic in detail. Operator build kits such as the Operator SDK use general RBAC defaults that developers may have not refined for their specific operator. Permissions afforded by the service account identity outside the cluster include federated and cross-cluster operators that have permissions in other Kubernetes clusters. As operators are increasingly used to manage off-cluster and cloud resources, cloud IAM integration permissions should be configured to prevent cloud account takeover from a compromised operator.·

·

_需要注意的一点_：特权的"圈地"——例如请求显著/管理权限——并不总是恶意的。开发者可能不太了解或者没有时间将所需权限调整到最小权限原则。即使是最无辜的情况，这仍然是一个红旗：也许 Operator 已经有足够的采用度让其他人发现并提出关于过度使用权限的担忧，也许这是 Operator 内部其他安全弱点的迹象。如果发现这样的"圈地"，建议谨慎行事。·

_One thing to note_: A "land grab" of privileges - e.g requesting significant/administrative access - is not always malicious in intent. The developer might not know better or have had the time to tune the required permissions to the concept of least privilege. Even in the most innocent case, though, it is still a red flag: Perhaps the operator has reached enough adoption for others to find and raise concerns about the overuse of privileges, and perhaps it is a sign of other security weaknesses within the operator. It is advisable to proceed with caution if such a "land grab" is found.·

·

**软件来源（Software provenance）**："软件供应链"在撰写本白皮书时开始受到更多关注。考虑 Operator 的来源、安装方式，以及恶意用户可能如何/为什么想要访问 Kubernetes 集群。在运行安装脚本之前花几分钟审查它。虽然 kubectl 命令支持直接从公共互联网应用 yaml 脚本（如 `kubectl create -f https://publicwebsite.com/install/operator.yaml`），但强烈建议先将该文件下载到本地，审查它，然后再运行 `kubectl create -f operator.yaml`。·

**Software provenance**: The "software supply chain" is starting to get more attention at the time of writing this whitepaper. Consider the source for an operator, how it is being installed, and how or why a malicious user may want access to a kubernetes cluster. Spend a few minutes reviewing an installation script before running it. While the kubectl command supports the ability to apply a yaml script directly from the public Internet (e.g `kubectl create -f https://publicwebsite.com/install/operator.yaml`) it is strongly recommended that one first downloads that file locally, review it, and then run `kubectl create -f operator.yaml`.·

·

审查脚本时请考虑以下问题：·

To review the script ask the following questions:·

·

- 此脚本的目的是什么？ What is the purpose of this script?·
- 脚本正在创建哪些资源？此脚本是否在创建 Role 和 RoleBinding？ What resources are being created by the script? Is this script creating Roles and RoleBindings?·
- 脚本将尝试使用哪些第三方来源？（如容器镜像、其他 yaml 文件）Git 和 Docker 镜像仓库有多流行和维护良好？这些可能是新项目的迹象、不再接收安全更新的废弃软件，或带有恶意意图的非官方仓库的指标。 What 3rd party sources will the script attempt to use? (e.g. container images, other yaml files) How popular and well-maintained are the git and docker image repositories? These might be signs of a new project, abandoned software which is no longer receiving security updates, or indicators of an unofficial repository with malicious intent.·
- 脚本试图获得什么权限？脚本是否尝试以主机共享或"特权模式"运行容器 securityContext？ What privileges does the script attempt to gain? Does the script attempt to run container securityContexts with host sharing or "privileged mode"?·

关于软件供应链安全的更多信息可在 [CNCF 供应链安全白皮书](https://github.com/cncf/tag-security/tree/main/community/working-groups/supply-chain-security/supply-chain-security-paper)中找到。

More information about software supply chain security is available in the [CNCF Supply Chain Security White Paper](https://github.com/cncf/tag-security/tree/main/community/working-groups/supply-chain-security/supply-chain-security-paper).

·

**高级安全控制**，如 SELinux、AppArmor 或 seccomp 可能由集群策略强制要求。开源 Operator 不太可能有这些 Linux 安全模块的配置，但如果组织熟悉其中一种控制系统，为 Operator 编写适当的安全配置不应需要大量的额外工作。·

**Advanced security controls**, such as SELinux, AppArmor, or seccomp may be mandated by cluster policy. Open source operators are unlikely to have configurations for these Linux security modules, but if an organization is familiar with one of these control systems, writing the appropriate security configuration for the operator should not require significant overhead.·

·

**Operator 配置**：理想情况下，项目应该是"默认安全"的，以增加安全 Operator 或应用部署的可能性。不安全的默认值需要手动配置来保护环境。虽然学习新 Operator 的配置参数似乎是不必要的工作，但通常比手动调整 Operator 本身的配置和/或源代码以达到所需的安全级别更可取。·

**Operator configuration**: Ideally a project will be "secure by default" to increase the likelihood of a secure operator or application deployment. Insecure defaults require manual configuration to secure the environment. While it may seem like unnecessary work to learn the configuration parameters of a new operator, it is usually preferable to manually adjusting the configuration and/or source code of an operator itself to reach the needed level of security.·

·

## 5. Kubernetes Operator 框架 Operator Frameworks for Kubernetes

目前存在许多框架来简化引导 Operator/控制器项目和编写 Operator 的过程。本章描述了其中一些，但不声称详尽无遗。·

Currently, many frameworks exist to simplify the process of bootstrapping an operator/controller project and to write operators. This chapter describes some of them without any claim to comprehensiveness.·

·

### 5.1 CNCF Operator Framework

*[Operator Framework](https://github.com/operator-framework)* 是一个开源工具包，以有效、自动化和可扩展的方式管理 Kubernetes 原生应用（称为 Operator）。·

The *[Operator Framework](https://github.com/operator-framework)* is an open source toolkit to manage Kubernetes native applications, called Operators, in an effective, automated, and scalable way.·

·

它面向 Operator 开发者，提供 SDK 来简化 Operator 开发，包含脚手架工具（基于 [kubebuilder](https://github.com/kubernetes-sigs/kubebuilder)）、单元测试和集成测试以及功能测试的测试工具，以及与用户可配置的更新图结合发布 Operator 版本历史的打包/分发机制。支持的项目类型有 Golang、Helm 和 Ansible。Python 和 Java 目前正在开发中。·

It aims at Operator Developers with an SDK to streamline Operator development with scaffolding tools (based on [kubebuilder](https://github.com/kubernetes-sigs/kubebuilder)), a test harness for unit tests and integration as well as functional tests and packaging / distribution mechanisms to publish version histories of Operators in conjunction with a user-configurable update graph. Supported project types are Golang, Helm and Ansible. Python and Java are currently in development.·

·

它也服务于 Kubernetes 管理员，他们需要在可能安装了数十个 Operator 的多租户环境中拥有安装、配置和更新 Operator 的中心点。它涵盖 Operator 生命周期的以下方面：·

It also caters for Kubernetes administrators that require a central point to install, configure and update Operators in a multi-tenant environment with potentially dozens of Operators installed. It covers the following aspects of Operator lifecycle:·

·

- 持续的空中更新和 Operator 目录，作为发布机制和更新来源 Continuous over-the-Air Updates and Catalogs of Operators as a publishing mechanism and source of updates·
- 依赖模型，使 Operator 可以依赖集群功能或相互依赖 Dependency Model so Operator can have dependencies on cluster features or on each other·
- 可发现性，使通常无法列出 CRD 或查看安装在单独命名空间中的 Operator 的低权限租户可以发现 Operator Discoverability for less privileged tenants that usually cannot list CRDs or see Operators installed in separate namespaces·
- 集群稳定性，避免多租户集群上 Operator 的运行时冲突，同时尊重 CRD 的全局性质和 CRD 版本控制及 CRD 转换的细微差别 Cluster Stability that avoid runtime conflicts of Operators on multi-tenant clusters while honoring the global nature of CRDs, and the subtleties of CRD versioning and CRD conversion·
- 声明式 UI 控制，允许控制台为与 Operator 服务交互的最终用户生成丰富的 UI 体验 Declarative UI controls that allows consoles to generate rich UI experiences for end users interacting with Operator services·

### 5.2 Kopf

**[Kopf](https://github.com/nolar/kopf)** ——**K**ubernetes **O**perator **P**ythonic **F**ramework——是一个用几行 Python 代码就能更快更简单地创建 Kubernetes Operator 的框架。它消除了大部分低级 Kubernetes API 通信的麻烦，将 Kubernetes 资源变更映射到 Python 函数并返回：·

**[Kopf](https://github.com/nolar/kopf)** —**K**ubernetes **O**perator **P**ythonic **F**ramework— is a framework to create Kubernetes operators faster and easier, just in a few lines of Python. It takes away most of the low-level Kubernetes API communication hassle and marshalls the Kubernetes resource changes to Python functions and back:·



```python
import kopf

@kopf.on.create(kind='KopfExample')
def created(patch, spec, **_):
    patch.status['name'] = spec.get('name', 'world')

@kopf.on.event(kind='KopfExample', field='status.name', value=kopf.PRESENT)
def touched(memo, status, **_):
    memo.last_name = status['name']

@kopf.timer('KopfExample', interval=5, when=lambda memo, **_: 'last_name' in memo)
def greet_regularly(memo, **_):
    print(f"Hello, {memo['last_name']}!")
```

如果你想要或需要在 Python 3.7+ 中创建临时（此处此刻、一次性、不可泛化的）Operator，你应该考虑使用此框架；特别是当你想将应用领域直接作为自定义资源引入 Kubernetes 时。·

You should consider using this framework if you want or need to make ad-hoc (here-and-now one-time non-generalizable) operators in Python 3.7+; especially if you want to bring your application domain directly to Kubernetes as custom resources.·

·

更多功能请参见[文档](https://kopf.readthedocs.io/en/stable/)。

For more features, see the [documentation](https://kopf.readthedocs.io/en/stable/).

·

### 5.3 kubebuilder

kubebuilder 框架为开发者提供了通过使用自定义资源定义来扩展 Kubernetes API 以及创建处理这些自定义资源的控制器的能力。·

The kubebuilder framework provides developers the possibilities to extend the Kubernetes API by using Custom Resource Definitions, and to create controllers that handle these custom resources.·

·

kubebuilder 框架提供的主要入口是一个 *Manager*。与原生 Kubernetes 控制器被分组到单个 Kubernetes 控制器管理器（`kube-controller-manager`）中的方式相同，你可以创建多个控制器并使它们由单个 Manager 管理。·

The main entry point provided by the kubebuilder framework is a *Manager*. In the same way the native Kubernetes controllers are grouped into a single Kubernetes Controller Manager (`kube-controller-manager`), you will be able to create several controllers and make them managed by a single manager.·

·

由于 Kubernetes API 资源附加到域并按 Group、Version 和 Kind 组织，你定义的 Kubernetes 自定义资源将附加到你自己的域，并按你自己的 Group、Version 和 Kind 组织。·

As Kubernetes API resources are attached to domains and arranged in Groups, Versions and Kinds, the Kubernetes custom resources you will define will be attached to your own domain, and arranged in your own groups, versions and kinds.·

·

使用 kubebuilder 的第一步是创建一个附加到你域的项目，这将创建构建单个 Manager 的源代码。·

The first step when using kubebuilder is to create a project attached to your domain, that will create the source code for building a single Manager.·

·

在用特定域初始化项目后，你可以向你的域添加 API 并使这些 API 由 Manager 管理。·

After initiating your project with a specific domain, you can add APIs to your domain and make these APIs managed by the manager.·

·

向项目添加资源将为你生成一些示例代码：一个你将调整以构建自己自定义资源的示例 *Custom Resource Definition*，以及一个将为处理此资源的 Operator 实现协调循环的示例 *Reconciler*。·

Adding a resource to the project will generate some sample code for you: a sample *Custom Resource Definition* that you will adapt to build your own custom resource, and a sample *Reconciler* that will implement the reconcile loop for your operator handling this resource.·

·

kubebuilder 框架利用了 `controller-runtime` 库，该库提供了 Manager 和 Reconciler 概念等。·

The kubebuilder framework leverages the `controller-runtime` library, that provides the Manager and Reconciler concepts, among others.·

·

kubebuilder 框架提供了构建 Manager 二进制文件、启动 Manager 的容器镜像以及部署此 Manager 所需的 Kubernetes 资源的所有必要条件，包括定义自定义资源的 `CustomResourceDefinition` 资源、部署 Manager 的 `Deployment`，以及 Operator 访问 Kubernetes API 所需的 RBAC 规则。·

The kubebuilder framework provides all the requisites for building the manager binary, the image of a container starting the manager, and the Kubernetes resources necessary for deploying this manager, including the `CustomResourceDefinition` resource defining your custom resource, a `Deployment` to deploy the manager, and RBAC rules for your operator to be able to access the Kubernetes API.·

·

### 5.4 Metacontroller - 轻量级 Kubernetes 控制器即服务 Metacontroller - Lightweight Kubernetes Controllers as a Service

[Metacontroller](https://metacontroller.github.io/metacontroller/) 是一个 Operator，使得编写和部署自定义 Operator 变得容易。·
[Metacontroller](https://metacontroller.github.io/metacontroller/) is an operator, that makes it easy to write and deploy custom operators.·

它自身引入了两个 CRD（2021年）：·

It introduces two CRD's itself (2021) :·

·

- [Composite Controller](https://metacontroller.github.io/metacontroller/api/compositecontroller.html) - 允许编写由 CRD 触发的 Operator [Composite Controller](https://metacontroller.github.io/metacontroller/api/compositecontroller.html) - allowing to write operator triggered by CRD·
- [Decorator Controller](https://metacontroller.github.io/metacontroller/api/decoratorcontroller.html) - 允许编写由任何 Kubernetes 对象（也包括由其他 Operator 管理的对象）触发的 Operator [Decorator Controller](https://metacontroller.github.io/metacontroller/api/decoratorcontroller.html) - allowing to write operator triggered by any kubernetes object (also managed by other operators)·

Metacontroller 本身，由其 CRD 之一配置，将负责观察集群状态并调用用户提供的控制器（用户控制器）来采取行动。·

Metacontrollers itself, configured by one of its CRD, will take care of observing cluster state and call controller, provided by user(user controller), to take actions.·

·

用户控制器应该以给定资源为输入，计算依赖对象的期望状态。·

User controller should, having given resources as input, compute the desired state of dependent objects.·

·

这也可以称为 `lambda 控制器` 模式（更多信息见[此处](https://metacontroller.github.io/metacontroller/concepts.html#lambda-controller)），因为输出仅根据输入计算，而 metacontroller 使用的逻辑也可以驻留在函数即服务（FaaS）提供商处。·

This could also be called `lambda controller` pattern (more on this [here](https://metacontroller.github.io/metacontroller/concepts.html#lambda-controller)), as the output is calculated only considering input and the logic used by metacontroller could also reside at a Function-as-a-Service provider.·

·

Metacontroller 的主要优势：·

Main advantages of metacontroller :·

·

- 只需提供一个函数（通过 webhook 调用），无需任何与监控 Kubernetes 资源相关的样板代码 Only a function (called via webhook) without any boilerplate related to watching kubernetes resources needs to be provided·
- 这样的函数可以用任何语言编写，并通过 HTTP 暴露 Such a function can be written in any language, and exposed via http·

主要限制：·

Main limitations :·

·

- 只能实现上述提到的某些模式 Only certain patterns are possible to implement, mentioned above·
- 当前架构依赖集群中的单个 metacontroller The current architecture relies on a single metacontroller in a cluster·
- Metacontroller 不感知任何外部状态，它完全依赖集群状态 Metacontroller is not aware of any external state, it relies entirely on cluster state·

以下示例 Metacontroller 配置用于为 `StatefulSet` 添加额外的网络暴露，而无需显式定义 `Service` 清单：·

Example metacontroller configuration, shown below, is used to add additional network exposure for `StatefulSet` without explicitly defining `Service` manifest.·



```yaml
apiVersion: metacontroller.k8s.io/v1alpha1
kind: DecoratorController
metadata:
  name: service-per-pod
spec:
  resources:
  - apiVersion: apps/v1
    resource: statefulsets
    annotationSelector:
      matchExpressions:
      - {key: service, operator: Exists}
      - {key: port, operator: Exists}
  attachments:
  - apiVersion: v1
    resource: services
  hooks:
    sync:
      webhook:
        url: http://service-per-pod.metacontroller/sync-service-per-pod
        timeout: 10s
```

使用上述配置：·

With above configuration :·

·

- `metacontroller` 对于匹配 `spec.resources` 描述的每个对象（本例中为带有 `service` 和 `port` 注解的 `apps/v1/statefulsets`），将监控匹配对象的任何变更（创建/更新/删除）并对每个对象调用 `hooks.sync` `metacontroller`, for every object matching `spec.resources` description (in this case - `apps/v1/statefulsets` with `service` and `port` annotations), will watch for any change in matching objects (create/update/delete) and invoke `hooks.sync` on each of those·
- `hooks.sync` 可以返回 `spec.attachments` 中描述的对象（本例中为 `v1/services`），这些对象将由 `metacontroller` 根据 `hook` 响应创建/更新/删除 the `hooks.sync` can return objects which are described in `spec.attachments` (in this case - `v1/services`) which will be created/updated/deleted by `metacontroller`, according to `hook` response·

例如，如果部署了以下 `StatefulSet`：·

For example, if below `StatefulSet` will be deployed:·



```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  annotations:
    service: "statefulset.kubernetes.io/pod-name"
    ports: "80:8080"
...
```

将创建以下 `Service` 对象：·

given `Service` object will be created by metacontroller:·



```yaml
apiVersion: "v1"
kind: "Service"
spec:
  selector: "statefulset.kubernetes.io/pod-name"
  ports:
  - port: 80
    targetPort: 8080
```

用户定义的端点（本例中为 `http://service-per-pod.metacontroller/sync-service-per-pod`）只需要关心 `Service` 的计算以及给定 `StatefulSet` 的 `Service` 应该是什么样子。·

The user defined endpoint (in this example - `http://service-per-pod.metacontroller/sync-service-per-pod`) only needs to care about the calculation of the `Service` and how it should look like for a given `StatefulSet`.·

·

使用 Metacontroller 可以实现的更多示例和想法，可以在 [metacontroller-examples](https://metacontroller.github.io/metacontroller/examples.html) 页面找到！·

Additional examples and ideas that could be implemented using metacontroller, can be found at the [metacontroller-examples](https://metacontroller.github.io/metacontroller/examples.html) page !·

·

如有任何问题，请访问我们的 Slack 频道（[#metacontroller](https://kubernetes.slack.com/archives/CA0SUPUDP)）或在 [GitHub Discussions](https://github.com/metacontroller/metacontroller/discussions/) 上提问。·

For any question, please visit our slack channel ([#metacontroller](https://kubernetes.slack.com/archives/CA0SUPUDP)) or ask it on [github discussions](https://github.com/metacontroller/metacontroller/discussions/).

·

## 6. Operator 生命周期管理 Operator Lifecycle Management

Operator 是一个应用程序，本节将描述有关 Operator 自身生命周期的考虑事项。·

An operator is an application, this section will describe considerations regarding the lifecycle of the operator itself.·

·

### 6.1 升级 Operator Upgrading the Operator

在升级 Operator 时，应特别注意被管理的资源。在 Operator 升级期间，被管理的资源应保持相同状态且健康。·

While upgrading the operator, special care should be taken in regards to the managed resources. During an operator upgrade, the managed resources should be kept in the same state and healthy.·

·

### 6.2 升级声明式状态 Upgrading the Declarative State

声明式状态是 Operator 的 API，它可能需要升级。CRD 版本的使用表示了 CRD 和 Operator 的稳定性——[阅读更多关于 CRD 版本控制的信息](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/)

The declarative state is the API of the operator, and it may need to be upgraded. The usage of CRD versions indicates the stability of the CRD and the operator - [read more about versioning a CRD](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/)

·

### 6.3 管理 CRD 的关系 Managing Relations of CRDs

随着 Operator 和 CRD 数量的增加，其管理复杂性也随之增加。例如，如何管理 Operator 之间的冲突，如两个与 Ingress 相关的功能？如何管理 CRD 之间的依赖关系和/或数据流关联，如数据库集群和数据库备份 CRD？·

As the number of Operators & CRDs adds up, its complexity of management also increases. For example, how to manage the conflicts between Operators, like two ingress-related functions? How to manage the dependencies and/or correlation of data flow between CRDs, like DB cluster and DB backup CRDs?·

·

为了解决这个问题，我们需要一个具体的模型来管理 Operator 和 CRD，以及一个新的基于策略的引擎机制来监督它们。社区的努力如 [KubeVela](https://kubevela.io/) 和 [Crossplane](https://crossplane.io/) 一直在尝试通过提供组合 CRD 的解决方案来解决这个问题。KubeVela 还提供了自定义资源之间数据依赖的管理。·

To resolve this problem, we would need a concrete model to manage Operators & CRDs and a new mechanism to oversee them with a policy-based engine. Community efforts like [KubeVela](https://kubevela.io/) and [Crossplane](https://crossplane.io/) have been trying to solve this problem by providing solutions to compose CRDs. KubeVela also provides management of data dependencies between custom resources.·

·

## 7. Operator 用例 Use Cases for an Operator

示例：Operator 用于安装应用或配置另一个对象，这是通过定义一组由 Operator 管理的对象以及它们如何相互工作来实现的。安装后，目标应用应无需人工干预即可运行。进一步地，控制器用于系统的重新配置。·

Example: An operator is used to install an application, or to provision another object which is achieved by defining a set of objects which are managed by the operator and how they work with each other. After the installation, the target application should be running without human interaction. In further consequence, a controller is used for the reconfiguration of a system.·

·

为实现这一点，Operator 监控当前状态和自定义资源或外部事件中做出的定义。比较它们并在需要时开始协调应用以达到期望状态。自定义资源中的变更可以是启用功能或更改版本，外部事件可以是 API 报告的应用更新可用性。当 Operator 管理的对象被删除时，应用的当前状态也可能不同，因此它们也会被重新创建以达到期望状态。·

To achieve this, an operator watches the current state and the definitions made in the custom resource or external events. Comparing them and starting to reconcile the application to get to the desired state when it is needed. Changes in the custom resource could be enabling a feature or changing a version, external events could be the availability of an application update reported by an API. The current state of the application could also differ when objects managed by the operator get deleted and so they also get recreated to get to the desired state.·

·

在更新应用时，Operator 包含达到新应用版本及如何过渡所需的逻辑。如上一章所述，这些可以是在更新数据库模式之前备份数据的机制。因此，Operator 中包含的逻辑知道构建一致备份需要哪些先决条件、如何备份数据以及如何恢复正常状态。·

When updating an application, the operator contains the logic which is needed to get to the new application version and how to transition. As described in the last chapter, these could be mechanisms to backup data before updating the database schema. Therefore, the logic included in the operator knows which prerequisites are necessary to build a consistent backup, how to backup the data and how to get back to the normal state.·

·

最后，Operator 能够移除应用及产生的对象。·

Finally, the operator is able to remove the application and the resulting objects.·

·

### 7.1 Prometheus Operator

Prometheus Operator 是有史以来最早编写的 Operator 之一（与 etcd 一起），它证明了这一问题空间的用例。·

The Prometheus Operator was one of the first ever Operators written, along with etcd, that proved the use case for this problem space.·

·

_"Prometheus Operator 旨在使在 Kubernetes 上运行 Prometheus 尽可能简单，同时保留 Kubernetes 原生的配置选项。"_·

_"The Prometheus Operator serves to make running Prometheus on top of Kubernetes as easy as possible, while preserving Kubernetes-native configuration options."_·

·

当安装 [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/developer/getting-started.md) 时，除了 Operator 控制器 Pod/Deployment 外，还会提供广泛的 API 来配置 Prometheus 技术栈。这些 API 以自定义资源定义（CRD）的形式表示，允许我们配置负责以下任务的对象：·

When the [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/developer/getting-started.md) is installed, besides the operator controller pod/deployment, a wide range of APIs becomes available to configure a Prometheus stack. The APIs are represented as Custom Resource Definitions (CRDs) which allow us to configure objects responsible, among other tasks, for:·

·

- 描述 Prometheus 监控的一组目标（ServiceMonitor）。 Describing a set of targets to be monitored by Prometheus (ServiceMonitor).·
- 声明式描述 Prometheus 部署的期望状态。 Declaratively describing the desired state of a Prometheus deployment.·
- 描述处理客户端应用发送的告警的 [AlertManager](https://github.com/prometheus/alertmanager) 集群。 Describing an [AlertManager](https://github.com/prometheus/alertmanager) cluster to handle alerts sent by client applications.·

好处是使用 Kubernetes 原生配置来配置整个运维技术栈，受益于 Kubernetes 资源验证和自愈能力。·

The benefit is using Kubernetes native configuration as a way to configure your whole operation stack, benefiting from Kubernetes resources validation and self-healing capabilities.·

·

然后 Operator 控制器将与 Kubernetes API 服务器通信，添加 Service 指标端点，并自动为配置的 Service 生成所需的 Prometheus 抓取配置。·

The Operator controller will then communicate with the Kubernetes API server to add Service metrics endpoints and automatically generate the required Prometheus scrape configurations for the configured Services.·

·

### 7.2 GitOps Operator Operator for GitOps

通常，Operator 与安装、升级和运维应用相关联。一个 Operator 也可以在不管理应用的情况下"运维"事物的例子可以在 GitOps 世界中找到。GitOps 是将 Git 作为所有资源的唯一真实来源的实践。·

Often, operators are associated with installing, upgrading and operating applications. One example that an operator could also "operate" things without managing an application can be found in the GitOps world. GitOps is the practice of using Git as the single source of truth for all resources.·

·

可能存在这样的情况：一个主要命令式管理的应用应该以更声明式和 Git 驱动的方式进行编排。因此，Operator 可以协助从 Git 仓库获取配置，分析配置以确定是否需要变更以及应采取哪些行动，并采取相应行动。·

There might be the case that an - mainly imperatively managed - application should be orchestrated in a more declarative and Git-driven way. Therefore, an operator could assist in fetching the configuration from a git-repository, analyze configurations to find out if something has to be changed and which actions should be taken and takes the according actions.·

·

![GitOps 示例 GitOps Example](img/071_GitOps_UseCase.png)

上面的示例说明了这种情况：·

The above example illustrates such a case:·

·

1. 一段配置被提交到 Git 仓库中。 a piece of configuration is checked in a git repository.·
2. Operator 通过使用自定义资源定义（其中存储了仓库路径和密钥信息）来确认 Git 仓库。 The operator acknowledges the git repository by using a custom resource definition (where the repository path and the information about the secret is stored).·
3. Operator 获取配置并进行分析。 The operator fetches the config and analyses it.·
4. 它应用其运维知识，从当前状态达到期望状态（通过查询应用的当前状态并发送指令以达到期望状态）。 It applies its operational knowledge to get from the current to the desired state (by querying the application about its current state and sending instructions to get to the desired state).·

这使用户能够拥有可重现的配置，并在 Git 仓库中进行版本控制。·

This enables the user to have reproducible configurations, versioned in a git repository.·

·

## 8. 成功模式 Successful Patterns

随着时间的推移，各种来源发布了大量编写 Operator 的最佳实践。下面将提及其中一些来源，并基于场景描述其中部分内容。·

Over time, lots of best practices for writing operators have been published by various sources. Following, some of these sources are mentioned and parts of them described based on a scenario.·

·

场景：一个微服务应用（"The PodTato Head"，https://github.com/cncf/podtato-head）应该完全通过 Operator 管理（即使另一种部署机制更合理）。此应用由 4 个服务和 1 个数据库组成，可如下所示：·

Scenario: A microservice application ("The PodTato Head", https://github.com/cncf/podtato-head) should be entirely managed via operators (even if another deployment mechanism would make more sense). This application consists of 4 services and 1 database which can be illustrated as follows:·

·

![示例应用 Sample Application](img/08_1_sample.png)

应将最佳实践应用于此应用部署。·

Best practices should be applied to this application deployment.·

·

### 8.1 管理单一类型的应用 Management of a single type of application

Operator 提供的功能应该特定于单一应用。应用到我们的示例中，这意味着应该有 5 个 Operator，每个管理一个组件（podtato-server、arm-service、foot-service、hat-service 和数据库）。这为所有组件提供了良好的关注点分离（基于 https://cloud.google.com/blog/products/containers-kubernetes/best-practices-for-building-kubernetes-operators-and-stateful-apps）。

The features an operator provides, should be specific to a single application. Applied to our example, this means that there should be 5 operators which will manage one component (podtato-server, arm-service, foot-service, hat-service and the database) at a time. This provides a good separation of concerns for all of them (based on https://cloud.google.com/blog/products/containers-kubernetes/best-practices-for-building-kubernetes-operators-and-stateful-apps).

·

### 8.2 Operator 的 Operator Operator of Operators

随着应用工作负载部署和管理生命周期中通常使用的 Operator 数量增长，在一组 Operator 之间出现了新的资源交互和元行为的机会。无论目标是减少管理多个异步执行资源变更的 Operator 的认知负担——还是确保发布版本之间的连续性；*Operator of Operators* 架构正在行业内的一些用例中得到应用。此范式通常利用 *Meta* Operator 创建多个资源，这些资源依次被异步创建，然后在元资源中更新。它使单一自定义资源定义能够表达期望的状态结果，并使需求能够被分区和异步执行。·

With a growing count of Operators typically used within the lifecycle of application workload deployment and management, there are opportunities for new interplay of resources and meta behaviors across a group of Operators. Whether the goal is to reduce the cognitive burden of managing multiple asynchronous Operators performing resource changes - or to ensure a level of continuity between release versions; the *Operator of Operators* architecture is being applied in some use cases within the industry. This paradigm typically utilizes a *Meta* Operator to create multiple resources that are in turn asynchronously created and then updated in the meta resource. It enables a single custom resource definition to express a desired state outcome and for the requirements to be partitioned and asynchronously acted upon.·

·

![分布式运维 Distributed Ops](img/09_1_distributedops.png)

协调整个技术栈的设置和生命周期可能仍然复杂。控制元数据资源的 Operator 可以通过协调技术栈的各个部分来帮助用户屏蔽这种复杂性，并暴露一个代表整个技术栈的 CRD。如果是这种情况，*Meta* Operator 应该将工作委托给其他 Operator 来处理更具体的部分。·

Coordinating the setup and lifecycle of the whole stack can remain complex. An Operator controlling a metadata resource can help shield the user from this complexity by coordinating the various parts of the stack and exposes a CRD representing the whole stack. If this is case, the *Meta* operator should delegate the work to the other Operators for the more specific parts.·

·

拥有这些技术栈子组件的控制器可以以两种方式出现：·

The controllers that own these sub-components of stacks can appear in two ways:·

·

- Operator 分发包可以由多个独立的控制器组成，每个控制器处理技术栈的一个子组件，加上一个主控制器（负责面向最终用户的 CRD，代表整个技术栈）。将这样的多控制器 Operator 作为单一包部署会导致所有控制器同时运行（每个一个 `Pod`），但只有面向最终用户的 API/CRD 被实际暴露和文档化供公众使用。当这种情况发生时，负责此 API 的控制器将若干职责委托给使用"内部" CRD 打包的其他控制器。当整个"技术栈"由同一组 Operator 作者拥有和开发且"从属"控制器作为独立项目没有意义时，这很有用。对最终用户来说，这组控制器仍然表现为单个 Operator。这里的主要好处是 Operator 项目内的关注点分离。 An operator distribution package could consist of multiple separate controllers, each handling a sub-component of the stack plus a main controller ( Responsible for the end-user facing CRD, representing the stack as a whole). Deploying such a multi-controller operator as a single package would result in all controllers running at once (one `Pod` each), but only the end-user facing API/CRD is actually exposed and documented for public consumption. When that happens, the controller responsible for this API delegates several duties to the other controllers, that are part of it's packaged using "internal" CRDs. This is useful when the whole "stack" is owned and developed by the same group of operator authors and the "subordinate" controllers don't make sense as a standalone project. To an end-user this set of controllers still appears as a single Operator. The main benefit here is separation of concerns within an operator project.·

![技术栈 Operator Stack-Operator](img/08_2_umbrella.png)

从技术上讲，会有一个由 Operator 管理的整个技术栈的自定义资源定义。此 Operator 为技术栈的每个组件创建一个自定义资源，这些组件又由 Operator 管理，管理底层资源。·

Technically, there would be a custom resource definition for the whole stack managed by an operator. This operator creates a custom resource for each of the components of the stack which are again managed by operators and managing the underlying resources.·

·

- 上面描述的第二种模式是高层工作负载 Operator。这些 Operator 依赖其他通用 Operator 项目来部署技术栈的子组件。例如，一个 Operator 依赖 `cert-manager`、`prometheus operator` 和 `postgresql` operator 来部署带有轮换证书、监控和 SQL 数据库的工作负载。在这种情况下，高层工作负载 Operator 不应试图在运行时安装 `cert-manager` 等。这是因为 Operator 作者随后要承担发布和维护这些依赖项的特定版本以及处理 CRD 生命周期管理的一般问题。 The second pattern depicted above, describes higher-level workload Operators. These depend on other general-purpose operator projects to deploy sub-components of a stack. An example would be an Operator, which depends on `cert-manager`, the `prometheus operator` and a `postgresql` operator to deploy its workload with rotating certificates, monitoring and a SQL database. In this case the higher-level workload operator should not try to ship and install `cert-manager` etc at runtime. This is because the operator author then signs up for shipping and maintaining the particular versions of these dependencies as well as dealing with the general problem area of CRD lifecycle management.·

*相反，应采用支持安装时依赖解析的包管理解决方案，这样安装其他所需 Operator 的工作就被委托给后台的包管理器，而不是作为高层 Operator 启动代码的一部分。*·

*Instead a package management solution should be employed that supports dependency resolution at install time, so that installing the other required operators is delegated to a package manager in the background and not as part of the higher level operator startup code.*·

·

这对于依赖其他独立有用且可能在集群上与多个其他 Operator 共享的 Operator 的 Operator 是有益的。[OLM](https://github.com/operator-framework/operator-lifecycle-manager) 是 Operator Framework 项目的一部分，就是这样一个包管理器。·

This is beneficial for operators that depend on other Operators, which are useful on their own and might even be shared with multiple other operators on the cluster. [OLM](https://github.com/operator-framework/operator-lifecycle-manager), part of the Operator Framework Project, is such a package manager.·

·

### 8.3 每个控制器一个 CRD One CRD per Controller

Operator 管理的每个 CRD 应该在单个控制器中实现。这使代码更易读，并有助于关注点分离。·

Every CRD managed by an operator should be implemented in a single controller. This makes code a bit more readable and should help with separation of concerns.·

·

### 8.4 在哪里发布和查找 Operator Where to publish and find Operators

有 operatorhub.io 和 artifacthub.io 等服务帮助最终用户查找 Operator，包括安装说明。这些服务通常包含当前安全问题和 Operator 来源的信息。此外，还提供了 Operator 能力的信息。·

There are services like operatorhub.io and artifacthub.io which help end-users to find operators including instructions on how they can be installed. These services often include information about current security issues and the sources of operators. Additionally, information about the capabilities of operators is given.·

·

### 8.5 延伸阅读 Further reading

还有更多最佳实践，如：·

There are lots of more best practices like:·

·

- Operator 不应安装其他 Operator An operator shouldn't install other operators·
- Operator 不应对其部署的命名空间做出假设 Operators shouldn't make assumptions about the namespaces they are deployed in, but also·
- 使用 SDK 编写 Operator Use an SDK for writing operators·

以及许多其他最佳实践可以在互联网上找到。更多内容可以在以下来源找到：·

and many other best practices might be found on the internet. More of them could be found on following sources:·

·

- https://github.com/operator-framework/community-operators/blob/master/docs/best-practices.md
- https://cloud.google.com/blog/products/containers-kubernetes/best-practices-for-building-kubernetes-operators-and-stateful-apps

## 9. 设计 Operator Designing Operators

前一章描述了一个最早 Operator 之一的用例。本章不声称详尽无遗，而是基于我们自己的经验或社区的描述，讨论编写自有 Operator 时的一些最佳实践。然而，如果没有对实际状态的清晰了解，也没有对我们要实现的目标的清晰想法，我们还需要一些方法和技术来指定 Operator 应该做什么。因此，我们还需要处理需求工程的某些方面。·

The previous chapter describes a use case for an operator that was one of the first operators ever. With no claim of completeness, this chapter deals with some best practices when writing own Operators, based on our own experience or described by the community. However, without clear knowledge of the actual state and without clear ideas of what we want to achieve, we also need some methods and techniques specifying what our Operator should do. Therefore, we will also have to deal with some aspects of requirement engineering.·

·

### 9.1 需求分析 Requirement Analysis

Kubernetes 的一个关键承诺是它能够在多个环境中以无（或最小）人工干预的方式自动化部署、扩缩容和管理容器化应用的运维任务。在 Kubernetes 中，无状态云原生应用非常适合水平扩缩容、自动自愈重启或新容器的渐进式推出。然而，在集群或分布式环境中运行的具有复杂组件的有状态应用并不总是适合这种基于容器的基础设施。在持久化、升级或高可用性方面，它们仍然需要人工交互才能保持稳定状态。·

A key promise of Kubernetes is that it enables the automation of operational tasks to deploy, scale, and manage containerized applications across multiple environments with no (or minimal) human intervention. In Kubernetes, stateless cloud native applications are well suited for horizontal scaling, automated self-healing restarts, or progressive rollout of new containers. However, stateful applications with complex components running in clustered or distributed environments are not always well suited for this type of container-based infrastructure. They still require human interaction when it comes to persistence, upgrades, or high availability to remain in a stable state.·

·

确实，Kubernetes 通过使用 Operator 创建和管理自定义应用以一种新颖的方式解决了这些问题。然而，这里有一个首要问题：作为开发者，你真的了解这类应用如何在内部和外部工作和交互吗？日常 IT 运维如何工作？应用如何备份（包括恢复）？在故障转移或停机时需要哪些步骤，软件组件之间是否存在依赖关系？·

True, Kubernetes solves these issues in a novel way by creating and managing custom applications using Operators. However, and here is the first question: as a developer, do you really know how this type of application works and interacts both internally and externally? How do the day-to-day IT operations work? How is the application backed up (including recovery)? What steps are necessary in case of failovers or outages, are there any dependencies between the software components?·

·

因此，强烈建议进行全面的需求分析来确定 Operator 的需求或条件。需求分析对 Operator 的成败至关重要。所有需求都应被记录、可衡量、可测试、可追溯、与已识别的需求相关，并在足够详细的层次上定义以支持系统设计。·

It is therefore strongly recommended that a comprehensive requirement analysis is needed to determine the requirements or conditions of an Operator. Requirement analysis is critical to the success or failure of Operators. All requirements should be documented, measurable, testable, traceable, related to identified requirements, and defined at a level of detail sufficient for system design.·

·

构建正确 Operator 的步骤：·

Steps to build the right operator:·

·

1. 如果不确定是否使用 Operator，请尝试运行可行性评估。找到使用 Operator 的合理且可理解的理由。将 Operator 的好处与实现和运维它们所需的努力进行对比。 If unsure whether to use an operator or not, try to run a feasibility assessment instead. Find plausible and understandable reasons for using an Operator. Contrast the benefits of Operators with the effort required to implement and operate them.·

2. 研究应用的现有文档，采访负责的系统管理员和其他利益相关者（如有必要），获取可能的系统检查活动列表、业务和 SLA 相关 KPI，并将它们与现有的事件报告或错误跟踪列表进行比较。 Study existing documentation of your application, interview responsible system administrators and other stakeholders (if necessary), get a list of possible system check activities, Business and SLA-relevant KPI and compare them with existing incident reports or bug tracking lists.·

3. 沿着"谁在何时做什么、怎么做、为什么"的思路，详细描述一个具体场景（如应用故障转移）。 Describe a concrete scenario (e.g., application failover) in detail along the lines of "who does what, when, how, and why".·

4. 描述 Operator 需要知道什么才能独立运行前述场景，使应用保持稳定和生产状态。 Describe what an Operator needs to know to run the previous scenario independently, keeping the application in a stable and productive state.·

### 9.2 自定义或第三方 Operator Custom or third-party Operator

既然使用 Operator 的情况已经明确，本文的下一部分将关注在哪里可以找到 Operator 实现以及哪个最符合需求。·

Now that the situations where using an Operator have been made clear, the next part of the paper will focus on where Operator implementations are available and which best meets requirements.·

·

找到合适的 Kubernetes Operator 可能是一项挑战。一方面，你需要找到符合你收集的需求的东西。另一方面，Operator 需要定期更新并得到供应商的积极支持。·

Finding the right Kubernetes Operator can be a challenge. On the one hand, you need to find something that fits with the requirements you have collected. On the other hand, the Operator needs to be regularly updated and actively supported by the vendor.·

·

简而言之，获取 Operator 有三种选择：·

In short, to get an Operator, you have three choices:·

·

(1) 你有一个数据库需要 Operator？咨询供应商的网站。 You have a database and need an Operator? Consult the website of the vendor.·

(2) 你可以搜索提供可用 Kubernetes Operator 的公共（或私有）注册中心。例如，[1] 提供了一个简化分发的 Operator 发布和共享平台。该平台使查找支持的服务和基本文档变得更容易。它还识别活跃的 Operator 社区和供应商支持的计划。 You can search for a public (or private) registry that offer available Kubernetes Operators. For example, [1] provides a platform for publishing and sharing Operators in a way that simplifies distribution. The platform makes it easier to find supported services and basic documentation. It also identifies active Operator communities and vendor-supported initiatives.·

(3) 自己编写 Operator，可以从零开始或使用合适的框架。 Write your own Operator, either from scratch or using a suitable framework.·

Operator 是特定于应用的，其功能范围从简单的安装脚本到处理升级、备份和故障的复杂逻辑。在公共注册中心找到合适的 Operator 需要时间和精力，代价可能是功能过大或缺失。相比之下，编写自定义 Operator 时，开发者想要或需要实现的功能没有限制，代价是开发和维护。·

Operators are application specific and their functionality ranges from a simple installation script to sophisticated logic that handles upgrades, backups and failures. It takes time and effort to find the right Operator in a public registry, at the cost of oversized or missing functionality. In contrast, when writing a custom Operator, there are no limits to the functionality developers want or need to implement, at the cost of development and maintenance.·

·

### 9.3 选择正确的工具 Use the right Tool

在完成完整的需求分析并决定编写自定义 Kubernetes Operator 之后，下一个问题是开发者应该使用哪些工具。[2] 的文章讨论了编写 Operator 的不同方法，并列出了每种解决方案的优缺点。该文章以一个 Operator 为例，使用各种技术和工具。具体来说，作者描述了以下工具：·

After completing and having a complete requirements analysis and deciding to write a custom Kubernetes Operator, the next question is which tools developers should use. The article by [2] discusses different approaches to writing Operators and lists the pros and cons of each solution. The article focuses on one Operator as an example and uses various techniques and tools. In detail, the author describes the following tools:·

·

(a) Operator SDK（Helm、Go、Ansible）。·
(a) Operator SDK (Helm, Go, Ansible).·

(b) Operator 框架 KOPF（Python）。 Operator framework KOPF (Python)·

(c) 裸编程语言（Java）。 Bare programming language (Java)·

如前所述，本文不仅描述了各个工具，还比较了它们的方法。作者证明命令式编程方法在开发过程中需要更多的时间、工作和谨慎。作为回报，它们给开发者提供编程任何所需逻辑的灵活性。相比之下，声明式方法（Helm Chart、Ansible）允许以非常简单的形式实现 Operator，既精确又可读。·

As mentioned earlier, this article not only describes the individual tools, but also compares their approaches. The author demonstrates that the imperative programming approaches require more time, work and caution during development. In return, they give developers the flexibility to program any kind of logic that is needed. In contrast, the declarative approaches (Helm Chart, Ansible) allow the implementation of Operators in a very simple form, which is precise and human-readable.·

·

[2] 的最佳实践是：·

Best practices of [2] are:·

·

1. 如果你**已经有 Helm chart** 且不需要复杂的能力级别 => Operator SDK: Helm If you **already have a Helm chart** for your software and you do not need any complex capability levels => Operator SDK: Helm·

2. 如果你想**快速创建 Operator** 且不需要复杂的能力级别 => Operator SDK: Helm If you want **to create your Operator quickly** and you do not need any complex capability levels => Operator SDK: Helm·

3. 如果你**想要复杂的功能**或/并希望对未来实现保持灵活 => Operator SDK: Go If you **want complex features** or/and be flexible about any future implementations => Operator SDK: Go·

4. 如果你希望在组织中保持**单一编程语言** If you want to keep a **single programming language in your organization**·

a. 如果你的语言有流行的 Operator 框架或/并你想为其做贡献 => Operator Framework·

a. If a popular Operator Framework exists for your language or/and you want to contribute to it => Operator Framework·

·

b. 如果你的编程语言没有流行的 Operator 框架 => 裸编程语言·

b. If no popular Operator Framework exists for your programming language => Bare Programming Language·

·

5. 如果**以上都不是** => Operator SDK: Go If **none of the above** => Operator SDK: Go·

### 9.4 选择正确的编程语言 Use the right programming language

Operator 是程序，可以用任何选择的语言编写。这是因为 Kubernetes 提供了 REST API，允许使用 HTTP 等轻量级协议与客户端通信。因此，软件开发者可以用他们喜欢的编程语言编写 Operator，只要遵循 REST API 规范。·

Operators are programs that can be written in any language of choice. This works because Kubernetes provides a REST API that allows communication with clients using lightweight protocols such as HTTP. Consequently, software developers can write Operators in their preferred programming language as long as the REST API specifications are followed.·

·

然而，如果开发者可以自由选择编程语言，迟早会出现不同技术和语言的拼凑。这将最终增加维护、故障排除、错误修复和支持请求的成本。更好的策略是专注于单一编程语言，并在团队中将其用于开发。这极大地支持了团队中的协作和互助。·

However, if developers are free to choose their programming language, sooner or later a patchwork of different technologies and languages will emerge. This will end up increasing costs for maintenance, troubleshooting, bug fixing and support requests. A better strategy is to focus on a single programming language and to use it for development as a team. This greatly supports the collaboration and mutual support in a team.·

·

然而，根据 [1]，**用 Go 语言编写的 Operator** 是最受欢迎的。原因有两方面：首先，Kubernetes 环境本身就是用 Go 编写的，因此客户端库得到了完美优化。其次，Operator SDK（内嵌 Kubebuilder）开箱即支持用 Go 实现 Operator。这为开发者节省了大量代码脚手架，并免费提供了代码生成。·

However, according to [1], **Operators written in Go Language** are by far the most popular. The reason for this is two-fold: first, the Kubernetes environment itself is written in Go, so the client library is perfectly optimized. Second, the Operator SDK (with embedded Kubebuilder) supports the implementation of Operators in Go out-of-the-box. This saves developers a lot of code scaffolding and gives them code generation for free.·

·

### 9.5 根据需求设计 Operator Design your Operator according to your needs

最后一段总结了各种来源发现和发布的最佳实践的无序列表。·

The last paragraph summarizes an unsorted list of best practices which were found and published by various sources.·

·

- 编写 Operator 涉及使用 Kubernetes API。使用 Operator-SDK 等框架可以节省时间，并获得一套工具来简化开发和测试。[3] Writing an Operator involves using the Kubernetes API. Use a framework like Operator-SDK to save yourself time with this and get a suite of tooling to ease development and testing. [3]

- 设计 Operator 时应确保即使 Operator 停止或移除，应用实例仍能不受影响地有效运行。 Design an Operator in such a way that application instance continues to run unaffected and effectively even if the Operator is stopped or removed.·

- 每个应用开发一个 Operator [4] Develop one Operator per application [4]

- Operator 应该向后兼容，并始终理解已创建资源的先前版本。 Operators should be backward compatible and always understand previous versions of resources that have already been created.·

- 使用异步同步循环 [4] Use asynchronous sync loops [4]

- Operator 应该利用内置的 Kubernetes 原语，如 ReplicaSet 和 Service。尽可能使用被充分理解和充分测试的代码。 Operators should leverage built-in Kubernetes primitives such as replica sets and services. Whenever possible, use well-understood and well-tested code.·

- 尽可能对 Operator 运行模拟 Pod、配置、存储和网络潜在故障的测试套件进行测试。 When possible, test Operators against a test suite that simulates potential failures of Pods, configuration, storage, and networking.·

### 9.6 参考文献 References

[1] https://operatorhub.io

[2] https://hazelcast.org/blog/build-your-kubernetes-operator-with-the-right-tool/

[3] https://github.com/operator-framework/community-operators/blob/master/docs/best-practices.md

[4] https://cloud.google.com/blog/products/containers-kubernetes/best-practices-for-building-kubernetes-operators-and-stateful-apps

## 10. 未来新兴模式 Emerging Patterns of the Future

随着 Operator 的流行度增加，出现了挑战最佳实践和设计原则现状的新用法和模式。·

As the popularity of Operators increases, there are new usages and patterns that are challenging the status-quo of best practices and design principles.·

·

### 10.1 Operator 生命周期管理 Operator Lifecycle Management

随着 Operator 复杂性的增加以及版本化的分布式控制器；对 Operator 及其资源的管理和透明度的需求也随之产生。此模式通过可发现性、最小依赖和声明式 UI 控制来帮助 Operator 的复用[1]。·

With increasing Operator complexity and versioned, distributed controllers; there has been a need for the management and transparency of Operators and their resources. This pattern aids in the reuse of Operators through discoverability, minimal dependencies and declarative UI controls[1].·

·

此外，随着 Operator 越来越被设计为以某些特征向预期终态进行协调，通过适当管理在集群内维护生命周期可以实现新行为的迭代、实验和测试。·

In addition to this, as Operators become increasingly designed to reconcile with certain characteristics toward an anticipated end-state, maintaining the life cycle within the cluster through proper management enables iterations, experimentation and testing of new behaviors.·

·

### 10.2 策略感知 Operator Policy-Aware Operators

许多 Operator 在集群内拥有一组静态的基于角色的授权来协调资源。目前有正在进行的活动为 Operator 提供更动态的访问，基于它们协调资源时需要表现的行为。这可能意味着临时提升权限以直接创建资源，或请求将自定义资源定义加载到 Kubernetes API 服务器中。·

Many Operators have a static set of role based authorizations within a cluster to reconcile resources. There is ongoing activity to provide operators more dynamic access, based on the behavior they are required to exhibit for reconciling a resource. This might mean a temporary elevation to create a resource directly, or to request that a custom resource definition is loaded into the Kubernetes API server.·

·

已有 Operator[2] 允许代表 Operator 特权创建资源的先例；并扩展到新的模式和运维模型[3]。此模式的未来潜力还将允许策略引擎控制 Operator 授权。·

There is precedent for Operators[2] to allow for privileged creation of resources on the behalf of the Operators; extending to new patterns and operating models[3]. Future potential of this pattern would also allow for a policy-engine to control Operator authorization.·

·

### 12.1 参考文献 References

[1] https://olm.operatorframework.io/

[2] https://github.com/cloud-ark/kubeplus

[3] https://oam.dev/

## 11. 结论 Conclusion

最初，Operator 是将状态应用接入通常处理无状态工作负载的编排器的一流解决方案。它们增强了编排器的 API 并进一步提升了容器编排器的能力，但没有解决应用配置和"Day 2"运维的所有问题。重要的是要记住，Operator 是一种管理特定需求和促进运维的模式，但它们也带来了复杂性，在实施之前应该进行权衡。·

Originally, operators were a first-class solution for onboarding stateful applications into orchestrators that usually tackled the operation of stateless workloads. They enhanced their APIs and increased the power of container orchestrators further but didn't resolve all problems of application configuration and "Day 2" operations. It is important to keep in mind that Operators are a pattern to manage specific requirements and facilitate operations but they also bring complexities that should be weighed up before being implemented.·

·

## 12. 相关工作 Related Work

最初，Operator 是通过 CoreOS 博客上的一篇博文引入的。该文章概述了 Operator 是什么、为什么要开发这个概念以及它们是如何构建的。本文中 Operator 的定义主要基于该文章的见解。由于博文只提供了简要概述，本文档更深入地描述了能力、安全等附加术语和其他概念。·

Initially, Operators were introduced by a blog post on the CoreOS Blog. This article provides a rough overview of what operators are, why the concept has been developed and how they are built. The insights of this article are mainly used for the definition of operators in this document. As the blog post only provided a concise overview, additional terms as capabilities, security and additional concepts are described more in-depth in this document.·

·

Operator 模式作为概念在 Kubernetes 文档中有所描述，因此提供了示例 Operator 可能如何工作的概述以及编写 Operator 的起点[1]。·

The Operator Pattern as a concept is described in the Kubernetes documentation and therefore provides an overview of how an example operator could do and provides starting points for writing an operator [1].·

·

《Kubernetes Operators》一书[2]提供了关于 Operator 的全面概述、它们解决的问题以及开发它们的不同方法。本书中的定义融入了本文档。《Kubernetes Patterns》（Ibryam, 2019）一书也是如此，它提供了对 Operator 更技术和概念性的洞察。这些书中的定义在本文档中进行了总结（以提供 Operator 的通用声明）。·

The Book "Kubernetes Operators" [2] provides a comprehensive overview about operators, which problems they solve and the different methods to develop them. Definitions made in this book flowed into this document. The same applies to the Book "Kubernetes Patterns" (Ibryam, 2019), which provides more technical and conceptual insights to operators. Definitions made in these books were summarized in this document (to provide a common declaration of operators).·

·

Michael Hausenblas 和 Stefan Schimanski [3] 写了一本关于《Programming Kubernetes》的书，提供了对 client-go、自定义资源以及编写 Operator 的更深入洞察。·

Michael Hausenblas and Stefan Schimanski [3] wrote a book about Programming Kubernetes, which provides deeper insights into client-go, custom resources, but also about writing operators.·

·

Google 发布了一篇关于构建 Kubernetes Operator 和有状态应用最佳实践的博文。该博文的一些建议出现在本白皮书的最佳实践部分[4]。·

Google provided a blog post about best practices for building Kubernetes Operators and stateful apps. Some of the advisories of this post take place in the best practices section of the whitepaper [4].·

·

许多文档描述了 Operator 的能力级别（也称为成熟度级别）。由于可能存在支持最高能力级别的所有功能但不支持某些较低级别功能的 Operator，本文档选择涵盖"能力"而非"能力级别"。不过，每个能力级别所需的能力仍被考虑在内[5]。·

Many documents describe capability levels (also known as maturity levels) of operators. Since there could be cases where an operator that supports all features that fall on the highest capability level but does not support some lower level features, this document chooses to cover "capabilities" rather than "capability levels". The capabilities required for each capability level, however, are taken into consideration [5].·

·

CNCF 安全 TAG 花费了大量精力为这篇白皮书添加安全相关主题。由于本白皮书的内容主要涵盖与 Operator 相关的安全措施，他们撰写了云原生安全白皮书，这是处理云原生安全时非常有用的资源[6]。·

The CNCF TAG Security spent a lot of effort to add security related topics to this whitepaper. As the content of this whitepaper should mostly cover operator-related security measures, they wrote a cloud native security whitepaper which is a very useful source when dealing with cloud native security [6].·

·

### 11.1 参考文献 References

[1] https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
[2] Dobies, J., & Wood, J. (2020). Kubernetes Operators. O'Reilly.·
[3] Michael Hausenblas and Stefan Schimanski, Programming Kubernetes: Developing Cloud-Native Applications, First edition. (Sebastopol, CA: O'Reilly Media, 2019).·
[4] https://cloud.google.com/blog/products/containers-kubernetes/best-practices-for-building-kubernetes-operators-and-stateful-apps
[5] Operator Framework. Retrieved 11 2020, 24, from https://operatorframework.io/operator-capabilities/, https://github.com/cloud-ark/kubeplus/blob/master/Guidelines.md
[6] https://github.com/cncf/tag-security/blob/main/community/resources/security-whitepaper/v2/cloud-native-security-whitepaper.md

## 13. 致谢 Acknowledgements

本文档是 CNCF TAG App-Delivery Operator 工作组社区驱动的工作成果。感谢所有为本文档做出贡献、参与讨论和审阅本文档的人。·

This document is a community-driven effort of the CNCF TAG App-Delivery Operator Working Group. Thanks to everyone who contributed to this document, joined discussions and reviewed this document.·

·

### 13.1 贡献者 Contributors

- Omer Kahani (github.com/OmerKahani)
- Jennifer Strejevitch (github.com/Jenniferstrej)
- Thomas Schuetz (github.com/thschue)
- Alex Jones (github.com/AlexsJones)
- Hongchao Deng (github.com/hongchaodeng)
- Grzegorz Głąb (github.com/grzesuav)
- Noah Kantrowitz (github.com/coderanger)
- John Kinsella (github.com/jlk)
- Philippe Martin (github.com/feloy)
- Daniel Messer (github.com/dmesser)
- Roland Pellegrini (github.com/friendlydevops)
- Cameron Seader (github.com/cseader)

### 13.2 审阅者 Reviewers

- Umanga Chapagain (github.com/umangachapagain)
- Michael Hrivnak (github.com/mhrivnak)
- Andy Jeffries (github.com/andyjeffries)
- Daniel Pacak (github.com/danielpacak)
- Bartlomiej Plotka (github.com/bwplotka)
- Phil Sautter (github.com/redeux)
- Roberth Strand (github.com/roberthstrand)
- Anais Urlichs (github.com/AnaisUrlichs)
