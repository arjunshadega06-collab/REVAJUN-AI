# REVAJUN-AI
Website AI buatan Arjun dan reva
export default async function handler(req, res) {
  if (req.method !== "POST") {
    return res.status(405).json({
      error: "Method tidak diizinkan"
    });
  }

  try {
    const { history, systemPrompt } = req.body;

    const response = await fetch(
      "https://api.openai.com/v1/responses",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${process.env.OPENAI_API_KEY}`
        },
        body: JSON.stringify({
          model: "gpt-6.2-astra",
          instructions: systemPrompt,
          input: history.map(item => ({
            role: item.role === "model" ? "assistant" : "user",
            content: item.text
          }))
        })
      }
    );

    const data = await response.json();

    if (!response.ok) {
      return res.status(response.status).json({
        error: data.error?.message || "API gagal"
      });
    }

    const reply = data.output
      ?.flatMap(item => item.content || [])
      ?.filter(item => item.type === "output_text")
      ?.map(item => item.text)
      ?.join("\n");

    return res.status(200).json({
      reply: reply || "Maaf, belum ada jawaban."
    });

  } catch (error) {
    return res.status(500).json({
      error: error.message
    });
  }
}
