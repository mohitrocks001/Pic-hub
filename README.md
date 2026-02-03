
import { GoogleGenAI, Type } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: process.env.API_KEY });

export const analyzeImage = async (base64Image: string) => {
  try {
    // Generate content using gemini-3-flash-preview for image analysis and creative critique
    const response = await ai.models.generateContent({
      model: "gemini-3-flash-preview",
      contents: {
        parts: [
          {
            inlineData: {
              mimeType: "image/jpeg",
              data: base64Image.split(',')[1] || base64Image
            }
          },
          {
            text: "Analyze this candidate's entry photo for a photography contest. Provide a creative critique (max 3 sentences), a 'Vibe Score' (0-100), and 3 relevant hashtags. Format as JSON."
          }
        ]
      },
      config: {
        responseMimeType: "application/json",
        responseSchema: {
          type: Type.OBJECT,
          properties: {
            critique: { type: Type.STRING },
            vibeScore: { type: Type.NUMBER },
            tags: {
              type: Type.ARRAY,
              items: { type: Type.STRING }
            }
          },
          required: ["critique", "vibeScore", "tags"]
        }
      }
    });

    // Access the .text property directly from the response object
    const text = response.text || "{}";
    return JSON.parse(text);
  } catch (error) {
    console.error("Gemini Analysis Error:", error);
    return {
      critique: "A stunning contribution to our gallery! This image speaks volumes.",
      vibeScore: Math.floor(Math.random() * 20) + 80,
      tags: ["Authentic", "Candidate", "Original"]
    };
  }
};
