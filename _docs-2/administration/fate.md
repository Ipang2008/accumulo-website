---
title: FATE
category: administration
order: 3
---

Accumulo must implement a number of distributed, multi-step operations to support
the client API. Creating a new table is a simple example of an atomic client call
which requires multiple steps in the implementation: get a unique table ID, configure
default table permissions, populate information in ZooKeeper to record the table's
existence, create directories in HDFS for the table's data, etc. Implementing these
steps in a way that is tolerant to node failure and other concurrent operations is
very difficult to achieve. Accumulo includes a Fault-Tolerant Executor (FATE) which
is widely used server-side to implement the client API safely and correctly.

Fault-Tolerant Executor (FATE) is the implementation detail which ensures that tables in
creation when the Manager dies will be successfully created when another Manager process is
started. This alleviates the need for any external tools to correct some bad state -- Accumulo
can undo the failure and self-heal without any external intervention.

## Overview

FATE consists of two primary components: a repeatable, persisted operation (REPO), a storage
layer for REPOs and an execution system to run REPOs. Accumulo uses ZooKeeper as the storage
layer for FATE and the Accumulo Manager acts as the execution system to run REPOs.

The important characteristic of REPOs are that they implemented in a way that is idempotent:
every operation must be able to undo or replay a partial execution of itself. Requiring the
implementation of the operation to support this functional greatly simplifies the execution
of these operations. This property is also what guarantees safety in light of failure conditions.

### REPO Stack

A FATE transaction is composed of a sequence of Repeatable persisted operations (REPO).  In order to start a FATE transaction,
a REPO is pushed onto a per-transaction REPO stack.  The top of the stack always contains the
next REPO the FATE transaction should execute.  When a REPO is successful it may return another
REPO which is pushed on the stack.

### FATE Structure in ZooKeeper

The storage layer in ZooKeeper is organized by storing each FATE transaction in a unique path based
on the FATE transaction id. The base path for FATE transactions is:

```
/accumulo/[INSTANCE_ID]/fate/tx_[TXID]
```

The data stored on the transaction id node provides the current FATE transaction status (e.g. NEW, IN_PROGRESS,
SUCCESS, FAILED,...)

Under the transaction id node, there will be a number of REPOs and a debug node that provides additional
information. The debug information is added when the transaction is created and is the command class simple name.
The REPOs form a stack of operations that will be performed in order and in ZooKeeper are numbered `repo_0000000000`
to `repo_#` The REPO with the largest number is the top of the stack. The top of the stack is the REPO currently
running or the next REPO that will start on the next execution. The REPO with the lowest number
(usually repo_0000000000) is the operation that spawned the FATE operations.

```
Sample FATE ZooKeeper paths:

/accumulo/dcbf6855-8eac-4b44-a4a9-7ad39caafe9a/fate/tx_4dd46d49d60f1a17
/accumulo/dcbf6855-8eac-4b44-a4a9-7ad39caafe9a/fate/tx_4dd46d49d60f1a17/debug
/accumulo/dcbf6855-8eac-4b44-a4a9-7ad39caafe9a/fate/tx_4dd46d49d60f1a17/repo_0000000002
/accumulo/dcbf6855-8eac-4b44-a4a9-7ad39caafe9a/fate/tx_4dd46d49d60f1a17/repo_0000000000

```

## Administration

Sometimes, it is useful to inspect the current FATE operations, both pending and executing.
For example, a command that is not completing could be blocked on the execution of another
operation. Accumulo provides an Accumulo shell command to interact with fate.

The `fate` admin command accepts a number of arguments for different functionality:
`list`/`print`, `summary`, `cancel`, `fail`, `delete`, `dump`.

The command for launching the fate admin command is:

```
> accumulo admin fate --[option]
```

The Accumulo admin help command option `accumulo admin -h` shows the expected usage information for the fate and
other admin commands.

### List/Print

Without any additional arguments, this command will print all operations that still exist in
the FATE store (ZooKeeper). This will include active, pending, and completed operations (completed
operations are lazily removed from the store). Each operation includes a unique "transaction ID", the
state of the operation (e.g. `NEW`, `SUBMITTED`, `IN_PROGRESS`, `FAILED`), any locks the
transaction actively holds and any locks it is waiting to acquire.

This option can also accept transaction IDs which will restrict the list of transactions shown.

### Summary (new in 2.1)

Similar to the List/Print command, this command prints a snapshot of all operations in the FATE store (ZooKeeper).
The information includes summary counts of:

  * Operation States (`NEW`, `SUBMITTED`, `IN_PROGRESS`, `FAILED`)
  * The FATE transaction commands
  * The current executing steps
  * Expanded FATE information details

The expanded FATE details supplement the information provided by list/print by including the running duration since the
FATE was created, the names of the namespace and tables for locks the transaction holds or is waiting to acquire.
(Note: depending on the operation and the step, the expanded details fields may be incomplete or unknown when the
snapshot information is gathered.)

This option accepts a filter for the details section by the state of the operation
(e.g. `NEW`, `SUBMITTED`, `IN_PROGRESS`, `FAILED`). The command also provides the option to output the information
formatted as json.

Sample output:

```
Report Time: 2022-07-07T11:42:02Z
Status counts:
  IN_PROGRESS: 2

Command counts:
  CompactRange: 2

Step counts:
  CompactionDriver: 2

Fate transactions (oldest first):
Status Filters: [NONE]

Running txn_id              Status      Command         Step (top)          locks held:(table id, name)             locks waiting:(table id, name)
0:00:04 0c143900c230c1df    IN_PROGRESS CompactRange    CompactionDriver    held:[R:(1,ns:ns1), R:(2,t:ns1.table1)] waiting:[]
0:00:03 55f59a2ae838e19e    IN_PROGRESS CompactRange    CompactionDriver    held:[R:(1,ns:ns1), R:(2,t:ns1.table1)] waiting:[]

```
### Cancel

This command can be used to cancel NEW or SUBMITTED FATE transactions. This command requires
one or more transaction ids.

### Fail

This command can be used to manually fail a FATE transaction and requires a transaction ID
as an argument. Failing an operation is not a normal procedure and should only be performed
by an administrator who understands the implications of why they are failing the operation.

### Delete

This command requires a transaction ID and will delete any locks that the transaction
holds. Like the fail command, this command should only be used in extreme circumstances
by an administrator that understands the implications of the command they are about to
invoke. It is not normal to invoke this command.

### Dump

This command accepts zero more transaction IDs.  If given no transaction IDs,
it will dump all active transactions.  A FATE operations is compromised as a
sequence of REPOs.  In order to start a FATE transaction, a REPO is pushed onto
a per-transaction REPO stack.  The top of the stack always contains the next
REPO the FATE transaction should execute.  When a REPO is successful it may
return another REPO which is pushed on the stack.  The `dump` command will
print all of the REPOs on each transactions stack.  The REPOs are serialized to
JSON in order to make them human-readable.
//This XML file does not appear to have any style information associated with it. The document tree is shown below.
<rss version="2.0">
<channel>
<title>MDN Blog</title>
<link>https://developer.mozilla.org/en-US/blog/</link>
<description>The MDN Web Docs blog publishes articles about web development, open source software, web platform updates, tutorials, changes and updates to MDN, and more.</description>
<lastBuildDate>Wed, 29 Nov 2023 00:29:54 GMT</lastBuildDate>
<docs>https://validator.w3.org/feed/docs/rss2.html</docs>
<generator>https://github.com/jpmonette/feed</generator>
<language>en</language>
<image>
<title>MDN Blog</title>
<url>https://developer.mozilla.org/mdn-social-share.png</url>
<link>https://developer.mozilla.org/en-US/blog/</link>
</image>
<copyright>All rights reserved 2023, MDN</copyright>
<item>
<title>
<![CDATA[ Getting started with CSS container queries ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/</link>
<guid>https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/</guid>
<pubDate>Thu, 16 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ CSS container queries are a powerful new tool for our CSS layout toolbox. In this post we'll dive into the practicalities of building a layout with container queries. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/css-container-queries.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Deploying Node.js applications with PM2 on Vultr ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/</link>
<guid>https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/</guid>
<pubDate>Wed, 08 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how to deploy a Node.js application on Vultr using PM2 to create persistent services. This guide shows how to efficiently use resources via PM2 cluster mode. It also covers Nginx server setup and SSL security. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ VS Code: Tips and tricks for beginners ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/</link>
<guid>https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/</guid>
<pubDate>Tue, 07 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Discover essential tips and tricks for using Visual Studio Code (VS Code), a powerful IDE. Learn how to leverage its integrated editing features and Git support, and explore a few extensions. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/vscode-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Coming Soon: MDN Observatory 2.0 ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/mdn-observatory/</link>
<guid>https://developer.mozilla.org/en-US/blog/mdn-observatory/</guid>
<pubDate>Wed, 25 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Observatory 2.0 is launching soon as part of the Mozilla Developer Network as the MDN Observatory with new security scoring standards and other exciting updates. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/mdn-observatory/mdn-observatory.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Optimizing DevSecOps workflows with GitLab's conditional CI/CD pipelines ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/</link>
<guid>https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/</guid>
<pubDate>Mon, 23 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ This guide explores the various types of CI/CD pipelines and helps you understand their specific use cases. Learn how to leverage rules to create highly efficient DevSecOps workflows. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introduction to web sustainability ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/</link>
<guid>https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/</guid>
<pubDate>Wed, 11 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ What can web designers and developers do to build a more sustainable web? This post explores the environmental impacts of web technologies and looks at some of the ways we can build greener websites. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/web-sustainability-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Migrating from GitHub to GitLab seamlessly: A step-by-step guide ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/</link>
<guid>https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/</guid>
<pubDate>Thu, 05 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Thinking about making the move from GitHub to GitLab? This guide demystifies the migration process, addressing common concerns for DevSecOps teams that are looking to seamlessly transition between the two platforms. This post provides a step-by-step guided tutorial on how to migrate your data from GitHub into GitLab. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Announcing the MDN front-end developer curriculum ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/</link>
<guid>https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/</guid>
<pubDate>Mon, 14 Aug 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN has created a curriculum for aspiring front-end developers to build a rewarding and successful career. Take a look at the curriculum, who it's for, and the research it's based on. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/mandala.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Creating custom easing effects in CSS animations using the linear() function ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/</link>
<guid>https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/</guid>
<pubDate>Tue, 01 Aug 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ The new CSS linear() timing function enables custom easing in animations. Explore how linear() works compared with other timing functions used for easing, with practical examples. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/linear-easing-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Securing your CDN: Why and how should you use SRI ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/</link>
<guid>https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/</guid>
<pubDate>Fri, 21 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Relying on external resources for your website is always fraught with risks. Learn how to protect your website and its visitors by using SRI to secure third-party content. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/sri.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Scroll progress animations in CSS ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/</link>
<guid>https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/</guid>
<pubDate>Fri, 14 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Scroll-driven animations are coming to CSS! In this post, we'll look at a few types of animations and learn how to link them to the scroll progress of a container. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/scroll-animations.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Reflections on AI Explain: A postmortem ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/</link>
<guid>https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/</guid>
<pubDate>Tue, 11 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ We recently launched a feature called AI Explain, but we have rolled this back for now. In this post, we look into the story behind AI Explain: its development, launch, and the reasons that led us to press the pause button. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/mandala.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Developer essentials: How to search code using grep ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/searching-code-with-grep/</link>
<guid>https://developer.mozilla.org/en-US/blog/searching-code-with-grep/</guid>
<pubDate>Mon, 03 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ grep is a powerful tool for searching code from the terminal. This post will show you how to use grep and why it's an essential developer tool. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/searching-code-with-grep/search-code-using-grep.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing AI Help (Beta): Your Companion for Web Development ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introducing-ai-help/</link>
<guid>https://developer.mozilla.org/en-US/blog/introducing-ai-help/</guid>
<pubDate>Tue, 27 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ We're introducing an AI assistant powered by MDN and OpenAI GPT 3.5 to answer all your web development questions in real time. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introducing-ai-help/mdn-ai-help.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Learn how to use hue in CSS colors with HSL ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/</link>
<guid>https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/</guid>
<pubDate>Mon, 26 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Hues are a bright way to define colors in CSS. Learn about hues, color wheels, how to use color functions, and how you can create vibrant color palettes for your website using hue. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/css-hues-colors.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing the MDN Playground: Bring your code to life! ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/</link>
<guid>https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/</guid>
<pubDate>Thu, 22 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN is launching a code Playground. Users can prototype ideas and expand all live samples into an interactive experience. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/play.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ MDN doc updates: CSS selectors & media queries, WebGPU & WebTransport APIs, Progressive web apps ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/</link>
<guid>https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/</guid>
<pubDate>Tue, 13 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Discover CSS :lang(), experimental media queries, manipulating graphics with WebGPU, client-server communication with WebTransport, ECMAScript module support, and more. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/mdn-june-2023.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ How to draw any regular shape with just one JavaScript function ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/</link>
<guid>https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/</guid>
<pubDate>Fri, 26 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how to use JavaScript to draw any regular shape to a HTML canvas with a single function, and how to modify it to draw multiple shapes. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/shape-drawing.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ New reference pages on MDN for JavaScript regular expressions ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/</link>
<guid>https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/</guid>
<pubDate>Tue, 23 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ See the latest updates to the MDN reference pages about JavaScript regular expressions, including new sections on sub-features and browser compatibility information. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/regex-reference-updates.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Celebrating Global Accessibility Awareness Day ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/</link>
<guid>https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/</guid>
<pubDate>Thu, 18 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ In celebration of Global Accessibility Awareness Day in 2023, we share some tools and guidelines to help you make the web more accessible. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/accessibility-awareness-day.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Using HTML landmark roles to improve accessibility ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/</link>
<guid>https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/</guid>
<pubDate>Mon, 15 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn what HTML landmark roles are, how they improve accessibility, and how you can include them on your website effectively. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/html-landmark-roles.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing Baseline: a unified view of stable web features ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/</link>
<guid>https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/</guid>
<pubDate>Wed, 10 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN leads the way in implementing WebDX community group's efforts, delivering a clear and simple baseline for the web platform to developers. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/baseline.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ How :not() chains multiple selectors ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/</link>
<guid>https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/</guid>
<pubDate>Fri, 05 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how the CSS `:not()` pseudo-class behaves when multiple selectors are passed as argument. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/css-not-pseudo-class.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ New functions, gradients, and hues in CSS colors (Level 4) ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/css-color-module-level-4/</link>
<guid>https://developer.mozilla.org/en-US/blog/css-color-module-level-4/</guid>
<pubDate>Wed, 03 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn what's new in CSS Colors Module Level 4, including color spaces, color functions, fancy gradients, and support for wide-gamut displays. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/css-color-module-level-4/css-color-functions-lvl4.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Welcome to the MDN blog ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/</link>
<guid>https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/</guid>
<pubDate>Wed, 03 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ The MDN blog publishes web development news, tutorials, and insights as an extension of MDN Web Docs, helping you discover, learn, and create for the web. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/mandala.png" length="0" type="image/png"/>
</item>
</channel>
</rss>/
//console.log(Float64Array.BYTES_PER_ELEMENT);
// Expected output: 8

console.log(Int8Array.BYTES_PER_ELEMENT);
// Expected output: 1
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
18947	City of Tipton - (IA)	8106	Tipton	Electric Utility	IA
12869	Monterey Regional Waste Mgmt	10748	Marina Landfill Gas	Commercial Non-CHP	CA
5109	DTE Electric Company	1740	River Rouge	Electric Utility	MI
8723	City of Holland	1830	James De Young	Electric Utility	MI
1009	City of Austin - (MN)	1961	Austin Northeast	Electric Utility	MN
18125	Stillwater Utilities Authority	3000	Boomer Lake Station	Electric Utility	OK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
221	Alaska Village Elec Coop, Inc	6319	Hooper Bay	Electric Utility	AK
50128	Georgia-Pacific Consr Ops LLC-Palatka	10611	Georgia-Pacific Palatka Operations	Industrial CHP	FL
17578	South Orange Co Wastewtr Auth	10820	Aliso Water Management Agency	Commercial CHP	CA
5701	El Paso Electric Co	55578	Hueco Mountain Wind Ranch	Electric Utility	TX
8198	City of Harrisonburg - (VA)	56006	Harrisonburg Power Plant	Electric Utility	VA
5517	Dynegy Midwest Generation Inc	898	Wood River	IPP Non-CHP	IL
5517	Dynegy Midwest Generation Inc	898	Wood River	IPP Non-CHP	IL
4161	Constellation Power Source Gen	1559	Riverside (MD)	IPP Non-CHP	MD
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
14165	NRG Power Midwest LP	2836	Avon Lake	IPP Non-CHP	OH
12807	Michigan South Central Pwr Agy	4259	Endicott Station	Electric Utility	MI
56217	Portsmouth Operating Services LLC	10071	Portsmouth Genco LLC	IPP Non-CHP	VA
56217	Portsmouth Operating Services LLC	10071	Portsmouth Genco LLC	IPP Non-CHP	VA
7726	Sharp Grossmont Hospital	10115	Grossmont Hospital	Commercial CHP	CA
7726	Sharp Grossmont Hospital	10115	Grossmont Hospital	Commercial CHP	CA
40211	Wabash Valley Power Assn, Inc	57842	Wabash Valley Power IGCC	Electric Utility	IN
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
12341	MidAmerican Energy Co	1091	George Neal North	Electric Utility	IA
12341	MidAmerican Energy Co	1091	George Neal North	Electric Utility	IA
5580	East Kentucky Power Coop, Inc	1385	Dale	Electric Utility	KY
5580	East Kentucky Power Coop, Inc	1385	Dale	Electric Utility	KY
4254	Consumers Energy Co	1695	B C Cobb	Electric Utility	MI
4254	Consumers Energy Co	1695	B C Cobb	Electric Utility	MI
4254	Consumers Energy Co	1720	J C Weadock	Electric Utility	MI
4254	Consumers Energy Co	1720	J C Weadock	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
5109	DTE Electric Company	1745	Trenton Channel	Electric Utility	MI
10000	Kansas City Power & Light Co	2080	Montrose	Electric Utility	MO
4045	City of Columbia - (MO)	2123	Columbia (MO)	Electric Utility	MO
15474	Public Service Co of Oklahoma	2963	Northeastern	Electric Utility	OK
17698	Southwestern Electric Power Co	6139	Welsh	Electric Utility	TX
20847	Wisconsin Electric Power Co	7549	Milwaukee County	Electric Utility	WI
26840	Port Townsend Paper Co	50544	Port Townsend Paper	Industrial CHP	WA
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	637	Rio Pinar	Electric Utility	FL
7801	Gulf Power Co	643	Lansing Smith	Electric Utility	FL
7801	Gulf Power Co	643	Lansing Smith	Electric Utility	FL
13168	NRG Huntley Operations Inc	2549	C R Huntley Generating Station	IPP Non-CHP	NY
13168	NRG Huntley Operations I
Int8Array.BYTES_PER_ELEMENT; // 1
Uint8Array.BYTES_PER_ELEMENT; // 1
Uint8ClampedArray.BYTES_PER_ELEMENT; // 1
Int16Array.BYTES_PER_ELEMENT; // 2
Uint16Array.BYTES_PER_ELEMENT; // 2
Int32Array.BYTES_PER_ELEMENT; // 4
Uint32Array.BYTES_PER_ELEMENT; // 4
Float32Array.BYTES_PER_ELEMENT; // 4
Float64Array.BYTES_PER_ELEMENT; // 8
BigInt64Array.BYTES_PER_ELEMENT; // 8
BigUint64Array.BYTES_PER_ELEMENT; // 8
Menyalin ke clipboard
Use the edit icon to pin, add or delete clips.254913168
new Int8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8ClampedArray([]).BYTES_PER_ELEMENT; // 1
new Int16Array([]).BYTES_PER_ELEMENT; // 2
new Uint16Array([]).BYTES_PER_ELEMENT; // 2
new Int32Array([]).BYTES_PER_ELEMENT; // 4
new Uint32Array([]).BYTES_PER_ELEMENT; // 4
new Float32Array([]).BYTES_PER_ELEMENT; // 429
new Float64Array([]).BYTES_PER_ELEMENT; // 8
new BigInt64Array([]).BYTES_PER_ELEMENT; // 9

new BigUint64Array([]This XML file does not appear to have any style information associated with it. The document tree is shown below.
<rss version="2.0">
<channel>
<title>MDN Blog</title>
<link>https://developer.mozilla.org/en-US/blog/</link>
<description>The MDN Web Docs blog publishes articles about web development, open source software, web platform updates, tutorials, changes and updates to MDN, and more.</description>
<lastBuildDate>Wed, 29 Nov 2023 00:29:54 GMT</lastBuildDate>
<docs>https://validator.w3.org/feed/docs/rss2.html</docs>
<generator>https://github.com/jpmonette/feed</generator>
<language>en</language>
<image>
<title>MDN Blog</title>
<url>https://developer.mozilla.org/mdn-social-share.png</url>
<link>https://developer.mozilla.org/en-US/blog/</link>
</image>
<copyright>All rights reserved 2023, MDN</copyright>
<item>
<title>
<![CDATA[ Getting started with CSS container queries ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/</link>
<guid>https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/</guid>
<pubDate>Thu, 16 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ CSS container queries are a powerful new tool for our CSS layout toolbox. In this post we'll dive into the practicalities of building a layout with container queries. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/css-container-queries.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Deploying Node.js applications with PM2 on Vultr ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/</link>
<guid>https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/</guid>
<pubDate>Wed, 08 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how to deploy a Node.js application on Vultr using PM2 to create persistent services. This guide shows how to efficiently use resources via PM2 cluster mode. It also covers Nginx server setup and SSL security. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ VS Code: Tips and tricks for beginners ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/</link>
<guid>https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/</guid>
<pubDate>Tue, 07 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Discover essential tips and tricks for using Visual Studio Code (VS Code), a powerful IDE. Learn how to leverage its integrated editing features and Git support, and explore a few extensions. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/vscode-featured.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Coming Soon: MDN Observatory 2.0 ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/mdn-observatory/</link>
<guid>https://developer.mozilla.org/en-US/blog/mdn-observatory/</guid>
<pubDate>Wed, 25 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Observatory 2.0 is launching soon as part of the Mozilla Developer Network as the MDN Observatory with new security scoring standards and other exciting updates. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/mdn-observatory/mdn-observatory.png" length="0" type="image/png"/>
</item>console.log(Int8Array.BYTES_PER_ELEMENT
<item>
<title>
<![CDATA[ Optimizing DevSecOps workflows with GitLab's conditional CI/CD pipelines ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/</link>
<guid>https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/</guid>
<pubDate>Mon, 23 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ This guide explores the various types of CI/CD pipelines and helps you understand their specific use cases. Learn how to leverage rules to create highly efficient DevSecOps workflows. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introduction to web sustainability ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/</link>
<guid>https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/</guid>
<pubDate>Wed, 11 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ What can web designers and developers do to build a more sustainable web? This post explores the environmental impacts of web technologies and looks at some of the ways we can build greener websites. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/web-sustainability-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Migrating from GitHub to GitLab seamlessly: A step-by-step guide ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/</link>
<guid>https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/</guid>
<pubDate>Thu, 05 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Thinking about making the move from GitHub to GitLab? This guide demystifies the migration process, addressing common concerns for DevSecOps teams that are looking to seamlessly transition between the two platforms. This post provides a step-by-step guided tutorial on how to migrate your data from GitHub into GitLab. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Announcing the MDN front-end developer curriculum ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/</link>
<guid>https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/</guid>
<pubDate>Mon, 14 Aug 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN has created a curriculum for aspiring front-end developers to build a rewarding and successful career. Take a look at the curriculum, who it's for, and the research it's based on. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/mandala.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Creating custom easing effects in CSS animations using the linear() function ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/</link>
<guid>https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/</guid>
<pubDate>Tue, 01 Aug 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ The new CSS linear() timing function enables custom easing in animations. Explore how linear() works compared with other timing functions used for easing, with practical examples. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/linear-easing-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Securing your CDN: Why and how should you use SRI ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/</link>
<guid>https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/</guid>
<pubDate>Fri, 21 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Relying on external resources for your website is always fraught with risks. Learn how to protect your website and its visitors by using SRI to secure third-party content. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/sri.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Scroll progress animations in CSS ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/</link>
<guid>https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/</guid>
<pubDate>Fri, 14 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Scroll-driven animations are coming to CSS! In this post, we'll look at a few types of animations and learn how to link them to the scroll progress of a container. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/scroll-animations.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Reflections on AI Explain: A postmortem ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/</link>
<guid>https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/</guid>
<pubDate>Tue, 11 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ We recently launched a feature called AI Explain, but we have rolled this back for now. In this post, we look into the story behind AI Explain: its development, launch, and the reasons that led us to press the pause button. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/mandala.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Developer essentials: How to search code using grep ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/searching-code-with-grep/</link>
<guid>https://developer.mozilla.org/en-US/blog/searching-code-with-grep/</guid>
<pubDate>Mon, 03 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ grep is a powerful tool for searching code from the terminal. This post will show you how to use grep and why it's an essential developer tool. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/searching-code-with-grep/search-code-using-grep.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Introducing AI Help (Beta): Your Companion for Web Development ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introducing-ai-help/</link>
<guid>https://developer.mozilla.org/en-US/blog/introducing-ai-help/</guid>
<pubDate>Tue, 27 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ We're introducing an AI assistant powered by MDN and OpenAI GPT 3.5 to answer all your web development questions in real time. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introducing-ai-help/mdn-ai-help.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Learn how to use hue in CSS colors with HSL ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/</link>
<guid>https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/</guid>
<pubDate>Mon, 26 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Hues are a bright way to define colors in CSS. Learn about hues, color wheels, how to use color functions, and how you can create vibrant color palettes for your website using hue. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/css-hues-colors.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing the MDN Playground: Bring your code to life! ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/</link>
<guid>https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/</guid>
<pubDate>Thu, 22 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN is launching a code Playground. Users can prototype ideas and expand all live samples into an interactive experience. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/play.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ MDN doc updates: CSS selectors & media queries, WebGPU & WebTransport APIs, Progressive web apps ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/</link>
<guid>https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/</guid>
<pubDate>Tue, 13 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Discover CSS :lang(), experimental media queries, manipulating graphics with WebGPU, client-server communication with WebTransport, ECMAScript module support, and more. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/mdn-june-2023.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ How to draw any regular shape with just one JavaScript function ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/</link>
<guid>https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/</guid>
<pubDate>Fri, 26 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how to use JavaScript to draw any regular shape to a HTML canvas with a single function, and how to modify it to draw multiple shapes. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/shape-drawing.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ New reference pages on MDN for JavaScript regular expressions ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/</link>
<guid>https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/</guid>
<pubDate>Tue, 23 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ See the latest updates to the MDN reference pages about JavaScript regular expressions, including new sections on sub-features and browser compatibility information. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/regex-reference-updates.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Celebrating Global Accessibility Awareness Day ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/</link>
<guid>https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/</guid>
<pubDate>Thu, 18 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ In celebration of Global Accessibility Awareness Day in 2023, we share some tools and guidelines to help you make the web more accessible. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/accessibility-awareness-day.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Using HTML landmark roles to improve accessibility ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/</link>
<guid>https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/</guid>
<pubDate>Mon, 15 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn what HTML landmark roles are, how they improve accessibility, and how you can include them on your website effectively. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/html-landmark-roles.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing Baseline: a unified view of stable web features ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/</link>
<guid>https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/</guid>
<pubDate>Wed, 10 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN leads the way in implementing WebDX community group's efforts, delivering a clear and simple baseline for the web platform to developers. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/baseline.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ How :not() chains multiple selectors ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/</link>
<guid>https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/</guid>
<pubDate>Fri, 05 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how the CSS `:not()` pseudo-class behaves when multiple selectors are passed as argument. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/css-not-pseudo-class.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ New functions, gradients, and hues in CSS colors (Level 4) ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/css-color-module-level-4/</link>
<guid>https://developer.mozilla.org/en-US/blog/css-color-module-level-4/</guid>
<pubDate>Wed, 03 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn what's new in CSS Colors Module Level 4, including color spaces, color functions, fancy gradients, and support for wide-gamut displays. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/css-color-module-level-4/css-color-functions-lvl4.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Welcome to the MDN blog ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/</link>
<guid>https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/</guid>
<pubDate>Wed, 03 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ The MDN blog publishes web development news, tutorials, and insights as an extension of MDN Web Docs, helping you discover, learn, and create for the web. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/mandala.png" length="0" type="image/png"/>
</item>
</channel>console.log(Int8Array.BYTES_PER_ELEMENT
</rss>.BYTES_PER_ELEMENT; // 89999
//Menyalin ke clipboard

new Int8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8ClampedArray([]).BYTES_PER_ELEMENT; // 1
new Int16Array([]).BYTES_PER_ELEMENT; // 2
new Uint16Array([]).BYTES_PER_ELEMENT; // 2
new Int32Array([]).BYTES_PER_ELEMENT; // 4
new Uint32Array([]).BYTES_PER_ELEMENT; // 4
new Float32Array([]).BYTES_PER_ELEMENT; // 4
new Float64Array([]).BYTES_PER_ELEMENT; // 8
new BigInt64Array([]).BYTES_PER_ELEMENT; // 8
new BigUint64Array([]).BYTES_PER_ELEMENT; // 8console.log(Float64Array.BYTES_PER_ELEMENT);
// Expected output: 8

console.log(Int8Array.BYTES_PER_ELEMENT);
// Expected output: 1
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
18947	City of Tipton - (IA)	8106	Tipton	Electric Utility	IA
12869	Monterey Regional Waste Mgmt	10748	Marina Landfill Gas	Commercial Non-CHP	CA
5109	DTE Electric Company	1740	River Rouge	Electric Utility	MI
8723	City of Holland	1830	James De Young	Electric Utility	MI
1009	City of Austin - (MN)	1961	Austin Northeast	Electric Utility	MN
18125	Stillwater Utilities Authority	3000	Boomer Lake Station	Electric Utility	OK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
221	Alaska Village Elec Coop, Inc	6319	Hooper Bay	Electric Utility	AK
50128	Georgia-Pacific Consr Ops LLC-Palatka	10611	Georgia-Pacific Palatka Operations	Industrial CHP	FL
17578	South Orange Co Wastewtr Auth	10820	Aliso Water Management Agency	Commercial CHP	CA
5701	El Paso Electric Co	55578	Hueco Mountain Wind Ranch	Electric Utility	TX
8198	City of Harrisonburg - (VA)	56006	Harrisonburg Power Plant	Electric Utility	VA
5517	Dynegy Midwest Generation Inc	898	Wood River	IPP Non-CHP	IL
5517	Dynegy Midwest Generation Inc	898	Wood River	IPP Non-CHP	IL
4161	Constellation Power Source Gen	1559	Riverside (MD)	IPP Non-CHP	MD
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
14165	NRG Power Midwest LP	2836	Avon Lake	IPP Non-CHP	OH
12807	Michigan South Central Pwr Agy	4259	Endicott Station	Electric Utility	MI
56217	Portsmouth Operating Services LLC	10071	Portsmouth Genco LLC	IPP Non-CHP	VA
56217	Portsmouth Operating Services LLC	10071	Portsmouth Genco LLC	IPP Non-CHP	VA
7726	Sharp Grossmont Hospital	10115	Grossmont Hospital	Commercial CHP	CA
7726	Sharp Grossmont Hospital	10115	Grossmont Hospital	Commercial CHP	CA
40211	Wabash Valley Power Assn, Inc	57842	Wabash Valley Power IGCC	Electric Utility	IN
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
12341	MidAmerican Energy Co	1091	George Neal North	Electric Utility	IA
12341	MidAmerican Energy Co	1091	George Neal North	Electric Utility	IA
5580	East Kentucky Power Coop, Inc	1385	Dale	Electric Utility	KY
5580	East Kentucky Power Coop, Inc	1385	Dale	Electric Utility	KY
4254	Consumers Energy Co	1695	B C Cobb	Electric Utility	MI
4254	Consumers Energy Co	1695	B C Cobb	Electric Utility	MI
4254	Consumers Energy Co	1720	J C Weadock	Electric Utility	MI
4254	Consumers Energy Co	1720	J C Weadock	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
5109	DTE Electric Company	1745	Trenton Channel	Electric Utility	MI
10000	Kansas City Power & Light Co	2080	Montrose	Electric Utility	MO
4045	City of Columbia - (MO)	2123	Columbia (MO)	Electric Utility	MO
15474	Public Service Co of Oklahoma	2963	Northeastern	Electric Utility	OK
17698	Southwestern Electric Power Co	6139	Welsh	Electric Utility	TX
20847	Wisconsin Electric Power Co	7549	Milwaukee County	Electric Utility	WI
26840	Port Townsend Paper Co	50544	Port Townsend Paper	Industrial CHP	WA
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	637	Rio Pinar	Electric Utility	FL
7801	Gulf Power Co	643	Lansing Smith	Electric Utility	FL
7801	Gulf Power Co	643	Lansing Smith	Electric Utility	FL
13168	NRG Huntley Operations Inc	2549	C R Huntley Generating Station	IPP Non-CHP	NY
13168	NRG Huntley Operations I
Int8Array.BYTES_PER_ELEMENT; // 1
Uint8Array.BYTES_PER_ELEMENT; // 1
Uint8ClampedArray.BYTES_PER_ELEMENT; // 1
Int16Array.BYTES_PER_ELEMENT; // 2
Uint16Array.BYTES_PER_ELEMENT; // 2
Int32Array.BYTES_PER_ELEMENT; // 4
Uint32Array.BYTES_PER_ELEMENT; // 4
Float32Array.BYTES_PER_ELEMENT; // 4
Float64Array.BYTES_PER_ELEMENT; // 8
BigInt64Array.BYTES_PER_ELEMENT; // 8
BigUint64Array.BYTES_PER_ELEMENT; // 8
Menyalin ke clipboard
Use the edit icon to pin, add or delete clips.254913168
new Int8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8ClampedArray([]).BYTES_PER_ELEMENT; // 1
new Int16Array([]).BYTES_PER_ELEMENT; // 2
new Uint16Array([]).BYTES_PER_ELEMENT; // 2
new Int32Array([]).BYTES_PER_ELEMENT; // 4
new Uint32Array([]).BYTES_PER_ELEMENT; // 4
new Float32Array([]).BYTES_PER_ELEMENT; // 429
new Float64Array([]).BYTES_PER_ELEMENT; // 8
new BigInt64Array([]).BYTES_PER_ELEMENT; // 9

new BigUint64Array([]This XML file does not appear to have any style information associated with it. The document tree is shown below.
<rss version="2.0">
<channel>
<title>MDN Blog</title>
<link>https://developer.mozilla.org/en-US/blog/</link>
<description>The MDN Web Docs blog publishes articles about web development, open source software, web platform updates, tutorials, changes and updates to MDN, and more.</description>
<lastBuildDate>Wed, 29 Nov 2023 00:29:54 GMT</lastBuildDate>
<docs>https://validator.w3.org/feed/docs/rss2.html</docs>
<generator>https://github.com/jpmonette/feed</generator>
<language>en</language>
<image>
<title>MDN Blog</title>
<url>https://developer.mozilla.org/mdn-social-share.png</url>
<link>https://developer.mozilla.org/en-US/blog/</link>
</image>
<copyright>All rights reserved 2023, MDN</copyright>
<item>
<title>
<![CDATA[ Getting started with CSS container queries ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/</link>
<guid>https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/</guid>
<pubDate>Thu, 16 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ CSS container queries are a powerful new tool for our CSS layout toolbox. In this post we'll dive into the practicalities of building a layout with container queries. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/css-container-queries.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Deploying Node.js applications with PM2 on Vultr ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/</link>
<guid>https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/</guid>
<pubDate>Wed, 08 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how to deploy a Node.js application on Vultr using PM2 to create persistent services. This guide shows how to efficiently use resources via PM2 cluster mode. It also covers Nginx server setup and SSL security. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ VS Code: Tips and tricks for beginners ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/</link>
<guid>https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/</guid>
<pubDate>Tue, 07 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Discover essential tips and tricks for using Visual Studio Code (VS Code), a powerful IDE. Learn how to leverage its integrated editing features and Git support, and explore a few extensions. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/vscode-featured.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Coming Soon: MDN Observatory 2.0 ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/mdn-observatory/</link>
<guid>https://developer.mozilla.org/en-US/blog/mdn-observatory/</guid>
<pubDate>Wed, 25 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Observatory 2.0 is launching soon as part of the Mozilla Developer Network as the MDN Observatory with new security scoring standards and other exciting updates. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/mdn-observatory/mdn-observatory.png" length="0" type="image/png"/>
</item>console.log(Int8Array.BYTES_PER_ELEMENT
<item>
<title>
<![CDATA[ Optimizing DevSecOps workflows with GitLab's conditional CI/CD pipelines ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/</link>
<guid>https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/</guid>
<pubDate>Mon, 23 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ This guide explores the various types of CI/CD pipelines and helps you understand their specific use cases. Learn how to leverage rules to create highly efficient DevSecOps workflows. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introduction to web sustainability ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/</link>
<guid>https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/</guid>
<pubDate>Wed, 11 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ What can web designers and developers do to build a more sustainable web? This post explores the environmental impacts of web technologies and looks at some of the ways we can build greener websites. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/web-sustainability-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Migrating from GitHub to GitLab seamlessly: A step-by-step guide ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/</link>
<guid>https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/</guid>
<pubDate>Thu, 05 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Thinking about making the move from GitHub to GitLab? This guide demystifies the migration process, addressing common concerns for DevSecOps teams that are looking to seamlessly transition between the two platforms. This post provides a step-by-step guided tutorial on how to migrate your data from GitHub into GitLab. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Announcing the MDN front-end developer curriculum ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/</link>
<guid>https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/</guid>
<pubDate>Mon, 14 Aug 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN has created a curriculum for aspiring front-end developers to build a rewarding and successful career. Take a look at the curriculum, who it's for, and the research it's based on. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/mandala.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Creating custom easing effects in CSS animations using the linear() function ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/</link>
<guid>https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/</guid>
<pubDate>Tue, 01 Aug 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ The new CSS linear() timing function enables custom easing in animations. Explore how linear() works compared with other timing functions used for easing, with practical examples. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/linear-easing-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Securing your CDN: Why and how should you use SRI ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/</link>
<guid>https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/</guid>
<pubDate>Fri, 21 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Relying on external resources for your website is always fraught with risks. Learn how to protect your website and its visitors by using SRI to secure third-party content. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/sri.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Scroll progress animations in CSS ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/</link>
<guid>https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/</guid>
<pubDate>Fri, 14 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Scroll-driven animations are coming to CSS! In this post, we'll look at a few types of animations and learn how to link them to the scroll progress of a container. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/scroll-animations.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Reflections on AI Explain: A postmortem ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/</link>
<guid>https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/</guid>
<pubDate>Tue, 11 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ We recently launched a feature called AI Explain, but we have rolled this back for now. In this post, we look into the story behind AI Explain: its development, launch, and the reasons that led us to press the pause button. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/mandala.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Developer essentials: How to search code using grep ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/searching-code-with-grep/</link>
<guid>https://developer.mozilla.org/en-US/blog/searching-code-with-grep/</guid>
<pubDate>Mon, 03 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ grep is a powerful tool for searching code from the terminal. This post will show you how to use grep and why it's an essential developer tool. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/searching-code-with-grep/search-code-using-grep.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Introducing AI Help (Beta): Your Companion for Web Development ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introducing-ai-help/</link>
<guid>https://developer.mozilla.org/en-US/blog/introducing-ai-help/</guid>
<pubDate>Tue, 27 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ We're introducing an AI assistant powered by MDN and OpenAI GPT 3.5 to answer all your web development questions in real time. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introducing-ai-help/mdn-ai-help.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Learn how to use hue in CSS colors with HSL ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/</link>
<guid>https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/</guid>
<pubDate>Mon, 26 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Hues are a bright way to define colors in CSS. Learn about hues, color wheels, how to use color functions, and how you can create vibrant color palettes for your website using hue. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/css-hues-colors.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing the MDN Playground: Bring your code to life! ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/</link>
<guid>https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/</guid>
<pubDate>Thu, 22 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN is launching a code Playground. Users can prototype ideas and expand all live samples into an interactive experience. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/play.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ MDN doc updates: CSS selectors & media queries, WebGPU & WebTransport APIs, Progressive web apps ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/</link>
<guid>https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/</guid>
<pubDate>Tue, 13 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Discover CSS :lang(), experimental media queries, manipulating graphics with WebGPU, client-server communication with WebTransport, ECMAScript module support, and more. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/mdn-june-2023.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ How to draw any regular shape with just one JavaScript function ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/</link>
<guid>https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/</guid>
<pubDate>Fri, 26 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how to use JavaScript to draw any regular shape to a HTML canvas with a single function, and how to modify it to draw multiple shapes. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/shape-drawing.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ New reference pages on MDN for JavaScript regular expressions ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/</link>
<guid>https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/</guid>
<pubDate>Tue, 23 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ See the latest updates to the MDN reference pages about JavaScript regular expressions, including new sections on sub-features and browser compatibility information. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/regex-reference-updates.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Celebrating Global Accessibility Awareness Day ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/</link>
<guid>https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/</guid>
<pubDate>Thu, 18 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ In celebration of Global Accessibility Awareness Day in 2023, we share some tools and guidelines to help you make the web more accessible. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/accessibility-awareness-day.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Using HTML landmark roles to improve accessibility ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/</link>
<guid>https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/</guid>
<pubDate>Mon, 15 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn what HTML landmark roles are, how they improve accessibility, and how you can include them on your website effectively. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/html-landmark-roles.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing Baseline: a unified view of stable web features ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/</link>
<guid>https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/</guid>
<pubDate>Wed, 10 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN leads the way in implementing WebDX community group's efforts, delivering a clear and simple baseline for the web platform to developers. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/baseline.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ How :not() chains multiple selectors ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/</link>
<guid>https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/</guid>
<pubDate>Fri, 05 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how the CSS `:not()` pseudo-class behaves when multiple selectors are passed as argument. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/css-not-pseudo-class.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ New functions, gradients, and hues in CSS colors (Level 4) ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/css-color-module-level-4/</link>
<guid>https://developer.mozilla.org/en-US/blog/css-color-module-level-4/</guid>
<pubDate>Wed, 03 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn what's new in CSS Colors Module Level 4, including color spaces, color functions, fancy gradients, and support for wide-gamut displays. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/css-color-module-level-4/css-color-functions-lvl4.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Welcome to the MDN blog ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/</link>
<guid>https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/</guid>
<pubDate>Wed, 03 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ The MDN blog publishes web development news, tutorials, and insights as an extension of MDN Web Docs, helping you discover, learn, and create for the web. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/mandala.png" length="0" type="image/png"/>
</item>
</channel>console.log(Int8Array.BYTES_PER_ELEMENT
</rss>.BYTES_PER_ELEMENT; // 89999
//Menyalin ke clipboard
console.log(Float64Array.BYTES_PER_ELEMENT);
// Expected output: 8

console.log(Int8Array.BYTES_PER_ELEMENT);
// Expected output: 1
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
20847	Wisconsin Electric Power Co	1784	Twin Falls (MI)	Electric Utility	MI
18947	City of Tipton - (IA)	8106	Tipton	Electric Utility	IA
12869	Monterey Regional Waste Mgmt	10748	Marina Landfill Gas	Commercial Non-CHP	CA
5109	DTE Electric Company	1740	River Rouge	Electric Utility	MI
8723	City of Holland	1830	James De Young	Electric Utility	MI
1009	City of Austin - (MN)	1961	Austin Northeast	Electric Utility	MN
18125	Stillwater Utilities Authority	3000	Boomer Lake Station	Electric Utility	OK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
4329	Copper Valley Elec Assn, Inc	6306	Valdez	Electric Utility	AK
221	Alaska Village Elec Coop, Inc	6319	Hooper Bay	Electric Utility	AK
50128	Georgia-Pacific Consr Ops LLC-Palatka	10611	Georgia-Pacific Palatka Operations	Industrial CHP	FL
17578	South Orange Co Wastewtr Auth	10820	Aliso Water Management Agency	Commercial CHP	CA
5701	El Paso Electric Co	55578	Hueco Mountain Wind Ranch	Electric Utility	TX
8198	City of Harrisonburg - (VA)	56006	Harrisonburg Power Plant	Electric Utility	VA
5517	Dynegy Midwest Generation Inc	898	Wood River	IPP Non-CHP	IL
5517	Dynegy Midwest Generation Inc	898	Wood River	IPP Non-CHP	IL
4161	Constellation Power Source Gen	1559	Riverside (MD)	IPP Non-CHP	MD
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
55768	RC Cape May Holdings LLC	2378	B L England	IPP Non-CHP	NJ
14165	NRG Power Midwest LP	2836	Avon Lake	IPP Non-CHP	OH
12807	Michigan South Central Pwr Agy	4259	Endicott Station	Electric Utility	MI
56217	Portsmouth Operating Services LLC	10071	Portsmouth Genco LLC	IPP Non-CHP	VA
56217	Portsmouth Operating Services LLC	10071	Portsmouth Genco LLC	IPP Non-CHP	VA
7726	Sharp Grossmont Hospital	10115	Grossmont Hospital	Commercial CHP	CA
7726	Sharp Grossmont Hospital	10115	Grossmont Hospital	Commercial CHP	CA
40211	Wabash Valley Power Assn, Inc	57842	Wabash Valley Power IGCC	Electric Utility	IN
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
18642	Tennessee Valley Authority	47	Colbert	Electric Utility	AL
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
9273	Indianapolis Power & Light Co	991	Eagle Valley (IN)	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
15470	Duke Energy Indiana, LLC	1010	Wabash River	Electric Utility	IN
12341	MidAmerican Energy Co	1091	George Neal North	Electric Utility	IA
12341	MidAmerican Energy Co	1091	George Neal North	Electric Utility	IA
5580	East Kentucky Power Coop, Inc	1385	Dale	Electric Utility	KY
5580	East Kentucky Power Coop, Inc	1385	Dale	Electric Utility	KY
4254	Consumers Energy Co	1695	B C Cobb	Electric Utility	MI
4254	Consumers Energy Co	1695	B C Cobb	Electric Utility	MI
4254	Consumers Energy Co	1720	J C Weadock	Electric Utility	MI
4254	Consumers Energy Co	1720	J C Weadock	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
4254	Consumers Energy Co	1723	J R Whiting	Electric Utility	MI
5109	DTE Electric Company	1745	Trenton Channel	Electric Utility	MI
10000	Kansas City Power & Light Co	2080	Montrose	Electric Utility	MO
4045	City of Columbia - (MO)	2123	Columbia (MO)	Electric Utility	MO
15474	Public Service Co of Oklahoma	2963	Northeastern	Electric Utility	OK
17698	Southwestern Electric Power Co	6139	Welsh	Electric Utility	TX
20847	Wisconsin Electric Power Co	7549	Milwaukee County	Electric Utility	WI
26840	Port Townsend Paper Co	50544	Port Townsend Paper	Industrial CHP	WA
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	629	G E Turner	Electric Utility	FL
6455	Duke Energy Florida, Inc	637	Rio Pinar	Electric Utility	FL
7801	Gulf Power Co	643	Lansing Smith	Electric Utility	FL
7801	Gulf Power Co	643	Lansing Smith	Electric Utility	FL
13168	NRG Huntley Operations Inc	2549	C R Huntley Generating Station	IPP Non-CHP	NY
13168	NRG Huntley Operations I
Int8Array.BYTES_PER_ELEMENT; // 1
Uint8Array.BYTES_PER_ELEMENT; // 1
Uint8ClampedArray.BYTES_PER_ELEMENT; // 1
Int16Array.BYTES_PER_ELEMENT; // 2
Uint16Array.BYTES_PER_ELEMENT; // 2
Int32Array.BYTES_PER_ELEMENT; // 4
Uint32Array.BYTES_PER_ELEMENT; // 4
Float32Array.BYTES_PER_ELEMENT; // 4
Float64Array.BYTES_PER_ELEMENT; // 8
BigInt64Array.BYTES_PER_ELEMENT; // 8
BigUint64Array.BYTES_PER_ELEMENT; // 8
Menyalin ke clipboard
Use the edit icon to pin, add or delete clips.254913168
new Int8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8ClampedArray([]).BYTES_PER_ELEMENT; // 1
new Int16Array([]).BYTES_PER_ELEMENT; // 2
new Uint16Array([]).BYTES_PER_ELEMENT; // 2
new Int32Array([]).BYTES_PER_ELEMENT; // 4
new Uint32Array([]).BYTES_PER_ELEMENT; // 4
new Float32Array([]).BYTES_PER_ELEMENT; // 429
new Float64Array([]).BYTES_PER_ELEMENT; // 8
new BigInt64Array([]).BYTES_PER_ELEMENT; // 9

new BigUint64Array([]This XML file does not appear to have any style information associated with it. The document tree is shown below.
<rss version="2.0">
<channel>
<title>MDN Blog</title>
<link>https://developer.mozilla.org/en-US/blog/</link>
<description>The MDN Web Docs blog publishes articles about web development, open source software, web platform updates, tutorials, changes and updates to MDN, and more.</description>
<lastBuildDate>Wed, 29 Nov 2023 00:29:54 GMT</lastBuildDate>
<docs>https://validator.w3.org/feed/docs/rss2.html</docs>
<generator>https://github.com/jpmonette/feed</generator>
<language>en</language>
<image>
<title>MDN Blog</title>
<url>https://developer.mozilla.org/mdn-social-share.png</url>
<link>https://developer.mozilla.org/en-US/blog/</link>
</image>
<copyright>All rights reserved 2023, MDN</copyright>
<item>
<title>
<![CDATA[ Getting started with CSS container queries ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/</link>
<guid>https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/</guid>
<pubDate>Thu, 16 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ CSS container queries are a powerful new tool for our CSS layout toolbox. In this post we'll dive into the practicalities of building a layout with container queries. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/getting-started-with-css-container-queries/css-container-queries.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Deploying Node.js applications with PM2 on Vultr ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/</link>
<guid>https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/</guid>
<pubDate>Wed, 08 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how to deploy a Node.js application on Vultr using PM2 to create persistent services. This guide shows how to efficiently use resources via PM2 cluster mode. It also covers Nginx server setup and SSL security. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/deploying-node-js-applications-with-pm2-on-vultr/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ VS Code: Tips and tricks for beginners ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/</link>
<guid>https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/</guid>
<pubDate>Tue, 07 Nov 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Discover essential tips and tricks for using Visual Studio Code (VS Code), a powerful IDE. Learn how to leverage its integrated editing features and Git support, and explore a few extensions. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/vs-code-tips-tricks/vscode-featured.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Coming Soon: MDN Observatory 2.0 ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/mdn-observatory/</link>
<guid>https://developer.mozilla.org/en-US/blog/mdn-observatory/</guid>
<pubDate>Wed, 25 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Observatory 2.0 is launching soon as part of the Mozilla Developer Network as the MDN Observatory with new security scoring standards and other exciting updates. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/mdn-observatory/mdn-observatory.png" length="0" type="image/png"/>
</item>console.log(Int8Array.BYTES_PER_ELEMENT
<item>
<title>
<![CDATA[ Optimizing DevSecOps workflows with GitLab's conditional CI/CD pipelines ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/</link>
<guid>https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/</guid>
<pubDate>Mon, 23 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ This guide explores the various types of CI/CD pipelines and helps you understand their specific use cases. Learn how to leverage rules to create highly efficient DevSecOps workflows. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/optimizing-devsecops-workflows-with-gitlab-conditional-ci-cd-pipelines/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introduction to web sustainability ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/</link>
<guid>https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/</guid>
<pubDate>Wed, 11 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ What can web designers and developers do to build a more sustainable web? This post explores the environmental impacts of web technologies and looks at some of the ways we can build greener websites. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/web-sustainability-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Migrating from GitHub to GitLab seamlessly: A step-by-step guide ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/</link>
<guid>https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/</guid>
<pubDate>Thu, 05 Oct 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Thinking about making the move from GitHub to GitLab? This guide demystifies the migration process, addressing common concerns for DevSecOps teams that are looking to seamlessly transition between the two platforms. This post provides a step-by-step guided tutorial on how to migrate your data from GitHub into GitLab. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/migrating-from-github-to-gitlab-seamlessly-a-step-by-step-guide/featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Announcing the MDN front-end developer curriculum ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/</link>
<guid>https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/</guid>
<pubDate>Mon, 14 Aug 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN has created a curriculum for aspiring front-end developers to build a rewarding and successful career. Take a look at the curriculum, who it's for, and the research it's based on. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/announcing-mdn-front-end-developer-curriculum/mandala.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Creating custom easing effects in CSS animations using the linear() function ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/</link>
<guid>https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/</guid>
<pubDate>Tue, 01 Aug 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ The new CSS linear() timing function enables custom easing in animations. Explore how linear() works compared with other timing functions used for easing, with practical examples. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/custom-easing-in-css-with-linear/linear-easing-featured.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Securing your CDN: Why and how should you use SRI ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/</link>
<guid>https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/</guid>
<pubDate>Fri, 21 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Relying on external resources for your website is always fraught with risks. Learn how to protect your website and its visitors by using SRI to secure third-party content. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/securing-cdn-using-sri-why-how/sri.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Scroll progress animations in CSS ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/</link>
<guid>https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/</guid>
<pubDate>Fri, 14 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Scroll-driven animations are coming to CSS! In this post, we'll look at a few types of animations and learn how to link them to the scroll progress of a container. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/scroll-progress-animations-in-css/scroll-animations.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Reflections on AI Explain: A postmortem ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/</link>
<guid>https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/</guid>
<pubDate>Tue, 11 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ We recently launched a feature called AI Explain, but we have rolled this back for now. In this post, we look into the story behind AI Explain: its development, launch, and the reasons that led us to press the pause button. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/ai-explain-postmortem/mandala.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Developer essentials: How to search code using grep ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/searching-code-with-grep/</link>
<guid>https://developer.mozilla.org/en-US/blog/searching-code-with-grep/</guid>
<pubDate>Mon, 03 Jul 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ grep is a powerful tool for searching code from the terminal. This post will show you how to use grep and why it's an essential developer tool. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/searching-code-with-grep/search-code-using-grep.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ Introducing AI Help (Beta): Your Companion for Web Development ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introducing-ai-help/</link>
<guid>https://developer.mozilla.org/en-US/blog/introducing-ai-help/</guid>
<pubDate>Tue, 27 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ We're introducing an AI assistant powered by MDN and OpenAI GPT 3.5 to answer all your web development questions in real time. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introducing-ai-help/mdn-ai-help.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Learn how to use hue in CSS colors with HSL ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/</link>
<guid>https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/</guid>
<pubDate>Mon, 26 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Hues are a bright way to define colors in CSS. Learn about hues, color wheels, how to use color functions, and how you can create vibrant color palettes for your website using hue. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/learn-css-hues-colors-hsl/css-hues-colors.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing the MDN Playground: Bring your code to life! ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/</link>
<guid>https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/</guid>
<pubDate>Thu, 22 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN is launching a code Playground. Users can prototype ideas and expand all live samples into an interactive experience. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/introducing-the-mdn-playground/play.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ MDN doc updates: CSS selectors & media queries, WebGPU & WebTransport APIs, Progressive web apps ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/</link>
<guid>https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/</guid>
<pubDate>Tue, 13 Jun 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Discover CSS :lang(), experimental media queries, manipulating graphics with WebGPU, client-server communication with WebTransport, ECMAScript module support, and more. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/mdn-docs-june-2023/mdn-june-2023.png" length="0" type="image/png"/>
</item>
<item>console.log(Int8Array.BYTES_PER_ELEMENT
<title>
<![CDATA[ How to draw any regular shape with just one JavaScript function ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/</link>
<guid>https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/</guid>
<pubDate>Fri, 26 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how to use JavaScript to draw any regular shape to a HTML canvas with a single function, and how to modify it to draw multiple shapes. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/javascript-shape-drawing-function/shape-drawing.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ New reference pages on MDN for JavaScript regular expressions ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/</link>
<guid>https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/</guid>
<pubDate>Tue, 23 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ See the latest updates to the MDN reference pages about JavaScript regular expressions, including new sections on sub-features and browser compatibility information. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/regular-expressions-reference-updates/regex-reference-updates.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Celebrating Global Accessibility Awareness Day ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/</link>
<guid>https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/</guid>
<pubDate>Thu, 18 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ In celebration of Global Accessibility Awareness Day in 2023, we share some tools and guidelines to help you make the web more accessible. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/accessibility-celebrating-gaad-2023/accessibility-awareness-day.png" length="0" type="image/png"/>
</item>
<item>
<title>console.log(Int8Array.BYTES_PER_ELEMENT
<![CDATA[ Using HTML landmark roles to improve accessibility ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/</link>
<guid>https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/</guid>
<pubDate>Mon, 15 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn what HTML landmark roles are, how they improve accessibility, and how you can include them on your website effectively. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/aria-accessibility-html-landmark-roles/html-landmark-roles.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Introducing Baseline: a unified view of stable web features ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/</link>
<guid>https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/</guid>
<pubDate>Wed, 10 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ MDN leads the way in implementing WebDX community group's efforts, delivering a clear and simple baseline for the web platform to developers. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/baseline-unified-view-stable-web-features/baseline.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ How :not() chains multiple selectors ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/</link>
<guid>https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/</guid>
<pubDate>Fri, 05 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn how the CSS `:not()` pseudo-class behaves when multiple selectors are passed as argument. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/css-not-pseudo-multiple-selectors/css-not-pseudo-class.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ New functions, gradients, and hues in CSS colors (Level 4) ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/css-color-module-level-4/</link>
<guid>https://developer.mozilla.org/en-US/blog/css-color-module-level-4/</guid>
<pubDate>Wed, 03 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ Learn what's new in CSS Colors Module Level 4, including color spaces, color functions, fancy gradients, and support for wide-gamut displays. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/css-color-module-level-4/css-color-functions-lvl4.png" length="0" type="image/png"/>
</item>
<item>
<title>
<![CDATA[ Welcome to the MDN blog ]]>
</title>
<link>https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/</link>
<guid>https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/</guid>
<pubDate>Wed, 03 May 2023 00:00:00 GMT</pubDate>
<description>
<![CDATA[ The MDN blog publishes web development news, tutorials, and insights as an extension of MDN Web Docs, helping you discover, learn, and create for the web. ]]>
</description>
<enclosure url="https://developer.mozilla.org/en-US/blog/welcome-to-the-MDN-blog/mandala.png" length="0" type="image/png"/>
</item>
</channel>console.log(Int8Array.BYTES_PER_ELEMENT
</rss>.BYTES_PER_ELEMENT; // 89999
//Menyalin ke clipboard

new Int8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8ClampedArray([]).BYTES_PER_ELEMENT; // 1
new Int16Array([]).BYTES_PER_ELEMENT; // 2
new Uint16Array([]).BYTES_PER_ELEMENT; // 2
new Int32Array([]).BYTES_PER_ELEMENT; // 4
new Uint32Array([]).BYTES_PER_ELEMENT; // 4
new Float32Array([]).BYTES_PER_ELEMENT; // 4
new Float64Array([]).BYTES_PER_ELEMENT; // 8
new BigInt64Array([]).BYTES_PER_ELEMENT; // 8
new BigUint64Array([]).BYTES_PER_ELEMENT; // 8
new Int8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8Array([]).BYTES_PER_ELEMENT; // 1
new Uint8ClampedArray([]).BYTES_PER_ELEMENT; // 1
new Int16Array([]).BYTES_PER_ELEMENT; // 2
new Uint16Array([]).BYTES_PER_ELEMENT; // 2
new Int32Array([]).BYTES_PER_ELEMENT; // 4
new Uint32Array([]).BYTES_PER_ELEMENT; // 4
new Float32Array([]).BYTES_PER_ELEMENT; // 4
new Float64Array([]).BYTES_PER_ELEMENT; // 8
new BigInt64Array([]).BYTES_PER_ELEMENT; // 8
new BigUint64Array([]).BYTES_PER_ELEMENT; // 8
