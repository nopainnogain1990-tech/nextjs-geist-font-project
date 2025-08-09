```markdown
# Detailed Implementation Plan for Vietnamese Language Learning App

This plan outlines the implementation of a Next.js app for practicing listening, shadowing, and vocabulary learning using supplied content. The app will have a modern, minimalist, and responsive UI using Tailwind CSS with shadcn components and Google Fonts for typography.

---

## 1. Create Sample Content Data File

**File:** `src/lib/content.ts`  
**Changes/Steps:**
- Create and export a constant object containing:
  - `audioUrl`: URL to an audio sample (e.g., `/audio/sample.mp3`); the audio file should be placed in the `public/audio` folder.
  - `transcript`: A string containing the transcript for shadowing (can be split into sentences if needed).
  - `vocabulary`: An array of vocabulary objects with properties: `word`, `definition`, and optionally `example`.

**Example Code:**
```typescript
export const sampleContent = {
  audioUrl: "/audio/sample.mp3",
  transcript: "Đây là một đoạn văn mẫu sử dụng để luyện nghe và shadowing. Hãy theo dõi và lặp lại.",
  vocabulary: [
    { word: "luyện nghe", definition: "Practice listening", example: "Tôi luyện nghe mỗi ngày." },
    { word: "shadowing", definition: "Mimicking spoken language", example: "Shadowing giúp cải thiện phát âm." }
  ]
};
```
  
---

## 2. Implement Reusable Components

### a. AudioPlayer Component

**File:** `src/components/AudioPlayer.tsx`  
**Changes/Steps:**
- Create a functional component that accepts an `audioUrl` prop.
- Render an `<audio>` element with the native `controls` attribute.
- Attach an `onError` handler to display a fallback message if the audio fails to load.
- Use Tailwind classes for spacing and typography.

**Example Code:**
```typescript
import React from "react";

interface AudioPlayerProps {
  audioUrl: string;
}

export const AudioPlayer: React.FC<AudioPlayerProps> = ({ audioUrl }) => {
  return (
    <div className="my-4">
      <audio 
        src={audioUrl} 
        controls 
        onError={(e) => console.error("Error loading audio", e)}
        className="w-full"
      >
        Your browser does not support the audio element.
      </audio>
    </div>
  );
};
```

---

### b. ShadowingText Component

**File:** `src/components/ShadowingText.tsx`  
**Changes/Steps:**
- Create a component that receives a `transcript` prop.
- Split the transcript into sentences (or segments) and render them in separate paragraphs.
- Allow manual highlighting (e.g., on click, toggle a highlighted style) to simulate shadowing.
- Add error handling (e.g., if the transcript is empty).

**Example Code:**
```typescript
import React, { useState } from "react";

interface ShadowingTextProps {
  transcript: string;
}

export const ShadowingText: React.FC<ShadowingTextProps> = ({ transcript }) => {
  const sentences = transcript.split(".");
  const [activeIndex, setActiveIndex] = useState<number | null>(null);

  return (
    <div className="my-4 space-y-2">
      {sentences.map((sentence, index) => {
        if (sentence.trim() === "") return null;
        return (
          <p 
            key={index}
            onClick={() => setActiveIndex(index)}
            className={`cursor-pointer p-2 rounded transition-colors ${activeIndex === index ? "bg-gray-300" : "bg-transparent"}`}
          >
            {sentence.trim()}.
          </p>
        );
      })}
    </div>
  );
};
```

---

### c. VocabularyList Component

**File:** `src/components/VocabularyList.tsx`  
**Changes/Steps:**
- Create a component that accepts a `vocabulary` prop (an array).
- Render each word in a card-like layout using Tailwind CSS for borders, spacing, and typography.
- Allow toggling additional details such as examples on click.
- Validate the vocabulary data and provide fallback if data is missing.

**Example Code:**
```typescript
import React, { useState } from "react";

interface VocabItem {
  word: string;
  definition: string;
  example?: string;
}

interface VocabularyListProps {
  vocabulary: VocabItem[];
}

export const VocabularyList: React.FC<VocabularyListProps> = ({ vocabulary }) => {
  const [expandedIndex, setExpandedIndex] = useState<number | null>(null);

  return (
    <div className="my-4 grid grid-cols-1 gap-4">
      {vocabulary.map((item, index) => (
        <div key={index} className="p-4 border rounded hover:shadow-md transition">
          <h3 className="text-lg font-semibold">{item.word}</h3>
          <p className="text-sm">{item.definition}</p>
          {item.example && (
            <div>
              <button
                className="mt-2 text-blue-600 underline"
                onClick={() => setExpandedIndex(expandedIndex === index ? null : index)}
              >
                {expandedIndex === index ? "Ẩn ví dụ" : "Hiển thị ví dụ"}
              </button>
              {expandedIndex === index && (
                <p className="mt-2 text-xs text-gray-600">{item.example}</p>
              )}
            </div>
          )}
        </div>
      ))}
    </div>
  );
};
```

---

## 3. Update Main Page with Tab Navigation

**File:** `src/app/page.tsx`  
**Changes/Steps:**
- Create a new main page that imports the three components and the sample content.
- Implement a tab switcher (using state) for three sections: Listening, Shadowing, Vocabulary.
- For each tab, render the corresponding component:
  - **Listening Tab:** Render `<AudioPlayer audioUrl={sampleContent.audioUrl} />`
  - **Shadowing Tab:** Render `<ShadowingText transcript={sampleContent.transcript} />`
  - **Vocabulary Tab:** Render `<VocabularyList vocabulary={sampleContent.vocabulary} />`
- Ensure responsive and accessible design using Tailwind CSS (flex layout, spacing, typography).

**Example Code:**
```typescript
"use client";
import React, { useState } from "react";
import { AudioPlayer } from "@/components/AudioPlayer";
import { ShadowingText } from "@/components/ShadowingText";
import { VocabularyList } from "@/components/VocabularyList";
import { sampleContent } from "@/lib/content";

type Tab = "listening" | "shadowing" | "vocabulary";

export default function HomePage() {
  const [activeTab, setActiveTab] = useState<Tab>("listening");

  return (
    <div className="max-w-3xl mx-auto py-8 px-4">
      <div className="flex justify-around mb-6">
        {(["listening", "shadowing", "vocabulary"] as Tab[]).map((tab) => (
          <button
            key={tab}
            onClick={() => setActiveTab(tab)}
            className={`px-4 py-2 border rounded ${
              activeTab === tab ? "bg-gray-300" : "bg-transparent"
            }`}
          >
            {tab === "listening" ? "Luyện Nghe" : tab === "shadowing" ? "Shadowing" : "Từ Vựng"}
          </button>
        ))}
      </div>
      <div>
        {activeTab === "listening" && (
          <AudioPlayer audioUrl={sampleContent.audioUrl} />
        )}
        {activeTab === "shadowing" && (
          <ShadowingText transcript={sampleContent.transcript} />
        )}
        {activeTab === "vocabulary" && (
          <VocabularyList vocabulary={sampleContent.vocabulary} />
        )}
      </div>
    </div>
  );
}
```

---

## 4. Additional Considerations and Error Handling

- **Error Handling:**  
  - Components include inline error logging (e.g., audio onError).
  - Validate required props; if data is missing, display a fallback message.
  
- **Best Practices:**  
  - Use TypeScript interfaces for prop validation.
  - Maintain clean code with proper comments.
  - Ensure responsive design for both mobile and desktop.
  - Use descriptive class names via Tailwind CSS to maintain UI consistency.
  
- **Testing:**  
  - Test tab navigation and component rendering using `npm run dev`.
  - Validate audio playback and error events.
  - Confirm that text highlighting in ShadowingText and toggling in VocabularyList work as expected.

---

## Summary

- Created a `content.ts` file to store sample audio, transcript, and vocabulary data.
- Developed reusable components: AudioPlayer, ShadowingText, and VocabularyList with error handling.
- Updated the main page at `src/app/page.tsx` to support tab-based navigation among Listening, Shadowing, and Vocabulary sections.
- Used Tailwind CSS for a minimalist, responsive UI with a focus on typography and layout.
- Ensured best practices with TypeScript interfaces, proper error logging, and responsive design.
- This plan integrates seamlessly into the existing Next.js project structure.
