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

EDU-Mate is a multimodal classroom learning assistant deployed on an Intel embedded platform. It combines live speech transcription, classroom images, structured notes, knowledge graphs, and an LLM-based Agent in one classroom workflow. During class, it records and organizes lecture content; after class, the same session becomes a searchable workspace for summaries, quizzes, review, and source-grounded question answering.

The public repository contains an earlier development snapshot. The final competition prototype was completed and integrated beyond the version currently uploaded.

<div style="width: 50%; max-width: 274px; margin-inline: auto;">
  {% include figure.liquid loading="eager" path="assets/img/projects/edu-mate/system-overview.png" title="EDU-Mate classroom use case" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  Classroom use case: EDU-Mate receives live audio and visual context from the teacher and turns it into an interactive learning workspace for the student.
</div>

## System architecture

{% include figure.liquid loading="eager" path="assets/img/projects/edu-mate/system-architecture.png" title="EDU-Mate system architecture" class="img-fluid rounded z-depth-1" %}

<div class="caption">
  System architecture used in the final design report. The Intel edge platform hosts the frontend, backend, local ASR, and local note-generation pipeline, while heavier multimodal and semantic tasks can be delegated through an OpenAI-compatible LLM interface.
</div>

The system is organized around a classroom `session`. Transcript segments, captured images, structured notes, knowledge-graph updates, Agent conversations, and post-class artifacts share the same session context, keeping information from different modalities aligned throughout a lecture.

Realtime and long-running AI tasks are deliberately separated. WhisperLive/OpenVINO handles streaming ASR locally, while a local Qwen model periodically turns stable transcripts into structured Markdown notes. Image understanding, knowledge extraction, classroom naming, and complex Agent responses can use cloud or locally hosted LLMs. Post-class tasks run asynchronously so ending a lecture is not blocked by long model calls.

## Agent and data workflow

I developed the central Agent pipeline that connects classroom data to downstream learning tasks. The backend does not simply forward the full transcript to an LLM. It maintains a structured classroom context and retrieves from stable transcript segments, Qwen-generated notes, image analyses, knowledge-graph nodes, and historical classroom material.

The Agent supports question answering, summaries, todo extraction, and quiz generation. A source-constrained mode grounds answers in retrieved classroom material and returns compact references and warnings when evidence is insufficient. The same retrieval layer also supports historical and cross-classroom review.

Knowledge extraction is integrated into the backend rather than treated as an external preprocessing stage. Transcript, note, and visual events can update the graph; duplicate or low-value concepts are filtered before graph patches are pushed to the frontend.

## Full-stack classroom application

<div style="width: 35%; margin-inline: auto;">
{% include figure.liquid path="assets/img/projects/edu-mate/classroom-test.jpg" title="EDU-Mate during classroom testing" class="img-fluid rounded z-depth-1" %}
</div>

<div class="caption">
  End-to-end classroom test: lecture content is shown on the classroom display while EDU-Mate simultaneously records subtitles, captures slides, and displays multimodal analysis in the local interface.
</div>

I implemented the FastAPI backend and React/TypeScript frontend that expose the Agent as a complete classroom application. The backend manages session lifecycle, event APIs, WebSocket updates, local persistence, RAG, LLM providers, knowledge extraction, Agent skills, and classroom history. Partial ASR results follow a preview-only path for low-latency display, while stable results enter the persistent transcript and retrieval pipeline.

The frontend provides the live classroom dashboard and the post-class review interface. Realtime and historical sessions reuse the same subtitle, image, graph, artifact, and Agent panels rather than maintaining separate application flows.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/edu-mate/realtime-subtitles.png" title="Realtime subtitle interface" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/edu-mate/image-analysis.png" title="Classroom image analysis" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="caption">
  Realtime subtitles preserve timing information, while captured classroom images are analyzed into descriptions, visible text, and key points that can later enter the knowledge graph and Agent retrieval pipeline.
</div>

## Knowledge organization and review

The project treats captured classroom content as reusable learning material rather than a collection of recordings and screenshots. Each session is stored with its transcript, structured notes, timeline, images, knowledge graph, summaries, todos, quizzes, and Agent history. This makes it possible to move from live lecture capture to post-class retrieval without rebuilding context.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/edu-mate/knowledge-graph.png" title="Knowledge graph generated from classroom content" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/edu-mate/agent-quiz.png" title="Agent-generated post-class quiz" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="caption">
  Classroom concepts are organized into an interactive knowledge graph, while the Agent can use saved classroom material to answer questions and generate review tasks such as self-tests.
</div>

## Result

The completed prototype was validated as an end-to-end classroom workflow rather than a collection of isolated AI demos. It continuously generated live subtitles, captured and interpreted lecture slides, organized concepts into an interactive graph, preserved classroom history, and converted the resulting material into summaries, quizzes, search results, and grounded Agent responses.

The main engineering problem was coordination: heterogeneous realtime inputs and long-running AI tasks had to share a consistent classroom state without interrupting live interaction. The final architecture uses session-scoped state, event-driven updates, asynchronous post-processing, and source-aware retrieval to connect lecture capture with post-class review.

## My contributions

- Designed and implemented the **Agent orchestration layer**, including session context, Agent skills, RAG retrieval, source-grounded responses, knowledge extraction, and local/cloud LLM integration.
- Developed the **FastAPI backend** for session lifecycle, realtime events, WebSocket updates, persistence, history, knowledge graphs, and asynchronous post-class processing.
- Developed the **React/TypeScript frontend** for live subtitles, classroom image capture and analysis, knowledge-graph visualization, Agent interaction, and historical review/search.
- Integrated the Agent workflow with streaming ASR, local Qwen note generation, multimodal classroom images, and the edge–cloud deployment pipeline.
- Led system-level integration and debugging of the Agent and full-stack application for the final competition prototype.
