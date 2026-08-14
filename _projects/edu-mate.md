---
layout: page
title: EDU-Mate
description: A multimodal classroom learning agent running on an Intel embedded platform.
img: assets/img/projects/edu-mate/system-overview.png
importance: 0
category: work
---

**2026 · Team project for the Intel Cup Undergraduate Electronic Design Contest – Embedded System Design Invitational Contest**  
**Role: Agent orchestration and full-stack development**

[Development snapshot on GitHub](https://github.com/ch-yu02/EDU-Mate)

EDU-Mate is a multimodal classroom learning assistant built for an Intel embedded platform. It combines live speech transcription, classroom images, structured notes, knowledge graphs, and an LLM-based classroom agent in one session-centered workflow. During class, it captures and organizes lecture content; afterward, the same session becomes a searchable workspace for summaries, quizzes, review, and source-grounded question answering.

The public repository contains an earlier development snapshot. The final competition prototype was completed and integrated beyond the version currently uploaded.

<div class="row mt-3">
  <div class="col-sm-3 mt-2">
    <div class="border rounded p-3 h-100 text-center">
      <strong>&gt; 200</strong><br>
      <span>subtitle segments in a classroom test</span>
    </div>
  </div>
  <div class="col-sm-3 mt-2">
    <div class="border rounded p-3 h-100 text-center">
      <strong>0.75 s</strong><br>
      <span>default preview cadence</span>
    </div>
  </div>
  <div class="col-sm-3 mt-2">
    <div class="border rounded p-3 h-100 text-center">
      <strong>&lt; 5 s</strong><br>
      <span>stable subtitle persistence in testing</span>
    </div>
  </div>
  <div class="col-sm-3 mt-2">
    <div class="border rounded p-3 h-100 text-center">
      <strong>&lt; 5 min</strong><br>
      <span>typical summary / todo generation</span>
    </div>
  </div>
</div>

## System architecture

<div style="width: 78%; margin-inline: auto;">
{% include figure.liquid loading="eager" path="assets/img/projects/edu-mate/system-architecture.png" title="EDU-Mate system architecture" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Edge–cloud system architecture.
</div>

The system is organized around a classroom `session`. Transcript segments, captured images, structured notes, knowledge-graph updates, agent conversations, and post-class artifacts all share the same session context, keeping information from different modalities aligned throughout a lecture.

Real-time work is separated from slower model calls. **WhisperLive with OpenVINO** handles streaming ASR locally, using `whisper-large-v3-turbo-fp16-ov` on the GPU/iGPU path. Stable transcripts are periodically reorganized by a **Qwen2.5-3B INT4** model running through OpenVINO GenAI on the CPU. Image understanding, knowledge extraction, classroom naming, and more complex agent responses can be delegated to cloud or locally hosted OpenAI-compatible LLMs.

The audio path is normalized to **16 kHz mono float32 PCM** before entering WhisperLive. In the default configuration, preview subtitles are pushed every **0.75 s**, transcript snapshots refresh every **5 s**, local Qwen notes update every **30 s**, and the optional cloud graph agent is rate-limited to one update every **60 s**.

> **Session flow:** live audio / images → structured classroom context → knowledge graph / retrieval → review and question answering

## Agent and data workflow

I developed the central agent pipeline that connects classroom data to downstream learning tasks. Rather than sending the full transcript directly to an LLM, the backend maintains a structured classroom context and retrieves from stable transcript segments, Qwen-generated notes, image analyses, knowledge-graph nodes, and historical classroom material.

The agent supports question answering, summaries, to-do extraction, and quiz generation. In source-constrained mode, answers are grounded in retrieved classroom material and return concise references together with warnings when the available evidence is insufficient. The same retrieval layer supports historical and cross-classroom review.

Knowledge extraction is part of the backend itself rather than a separate preprocessing stage. Transcript, note, and visual events can update the graph, while duplicate and low-value concepts are filtered before incremental graph patches are pushed to the frontend.

## Full-stack classroom application

<div style="width: 38%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/edu-mate/classroom-test.jpg" title="EDU-Mate classroom test" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  End-to-end classroom test.
</div>

I implemented the FastAPI backend and React/TypeScript frontend that turn the agent pipeline into a complete classroom application. The backend manages session lifecycle, event APIs, WebSocket updates, local persistence, RAG, LLM providers, knowledge extraction, agent skills, and classroom history. Partial ASR output stays on a preview-only path for low-latency display; only stable segments enter the persistent transcript and retrieval pipeline.

The frontend provides both the live classroom dashboard and the post-class review interface. Real-time and historical sessions reuse the same subtitle, image, graph, artifact, and agent panels rather than maintaining separate application flows.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/edu-mate/realtime-subtitles.png" title="Realtime subtitles" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/edu-mate/image-analysis.png" title="Classroom image analysis" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="caption">
  Live subtitles and visual analysis.
</div>

In the reported classroom test, the system accumulated **171 timestamped English subtitle segments** while following a computer-networking lecture. Stable subtitle results were saved within **5 seconds** in the observed test flow. The visual pipeline captured projected slides, correctly identified topics such as _Internet_, _protocols_, and _Internet standards_, and converted each processed image into a caption, visible text, key points, and optional graph entities.

Visual analysis also has explicit failure handling. While a session is still recording, failed image analyses can be retried **up to two times at 4-second intervals**. The backend increases the LLM timeout by **80% after each failure**, capped at **300 seconds**, so unusually complex images are given more time without making every request slow by default.

## Knowledge organization and review

The project treats classroom recordings as reusable learning material rather than a collection of disconnected transcripts and screenshots. Each session stores its transcript, structured notes, timeline, images, knowledge graph, summaries, to-do items, quizzes, and agent history. That continuity makes it possible to move from live capture to post-class retrieval without reconstructing context.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/edu-mate/knowledge-graph.png" title="Knowledge graph" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/edu-mate/agent-quiz.png" title="Agent-generated quiz" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="caption">
  Knowledge graph and generated quiz.
</div>

The final knowledge graph from the classroom test connected concepts including **protocol, HTTP, TCP, Ethernet, Wi-Fi, IP, 4G, and bandwidth**. The graph supports list and graph views, zooming, dragging, fullscreen inspection, and source-aware nodes and relations. The agent used the same saved material to generate self-test questions tied to the lecture rather than producing generic open-domain questions.

Ending a class is intentionally non-blocking. The frontend uses a **15-second** request timeout for the end-class operation; the backend first persists the core session and then continues summary, to-do, final extraction, and optional indexing work asynchronously. In testing, **summary and to-do generation typically completed within five minutes**, depending on lecture length, model latency, and network conditions.

## Edge deployment details

The prototype was designed around an Intel-based embedded platform rather than a cloud-only stack:

- **ASR:** WhisperLive + OpenVINO, with GPU/iGPU preferred for `whisper-large-v3-turbo-fp16-ov`.
- **Local note generation:** Qwen2.5-3B INT4 through OpenVINO GenAI on CPU.
- **Audio front end:** a four-microphone directional array; the module specification lists approximately **1.5 m** directional pickup, **24-bit / 16 kHz** audio, **26 dB** typical SNR, and up to **20 dB** noise reduction when input SNR exceeds 5 dB.
- **Visual capture:** browser camera path configured for an ideal **1280 × 720** frame before upload and multimodal analysis.
- **Storage:** session files remain local, including transcript, timeline, structured notes, images, graph, summary, todos, quiz, and agent artifacts.

This division keeps the latency-sensitive capture path local while reserving heavier semantic reasoning for models that can be swapped between cloud and local OpenAI-compatible providers.

## Result

The completed prototype was validated as an end-to-end classroom system rather than a set of isolated AI demonstrations. It continuously produced live subtitles, captured and interpreted lecture slides, organized concepts into an interactive graph, preserved classroom history, and turned that material into summaries, quizzes, search results, and grounded agent responses.

The central engineering challenge was coordination: heterogeneous real-time inputs and long-running AI tasks had to share a consistent classroom state without disrupting live interaction. The final architecture uses session-scoped state, event-driven updates, asynchronous post-processing, and source-aware retrieval to connect lecture capture with post-class review.

## My contributions

- Designed and implemented the **agent orchestration layer**, including session context, agent skills, RAG retrieval, source-grounded responses, knowledge extraction, and local/cloud LLM integration.
- Developed the **FastAPI backend** for session lifecycle, real-time events, WebSocket updates, persistence, history, knowledge graphs, and asynchronous post-class processing.
- Developed the **React/TypeScript frontend** for live subtitles, classroom image capture and analysis, knowledge-graph visualization, agent interaction, and historical review/search.
- Integrated the agent workflow with streaming ASR, local Qwen note generation, multimodal classroom images, and the edge-cloud deployment pipeline.
- Led system-level integration and debugging of the agent and full-stack application for the final competition prototype.
