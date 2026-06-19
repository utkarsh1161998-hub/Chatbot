import os
import anthropic

def run_chatbot():
    """Simple terminal chatbot using the Anthropic API."""
    client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

    conversation_history = []
    system_prompt = "You are a helpful, friendly assistant. Be concise and clear."

    print("=" * 50)
    print("  Claude Chatbot  ")
    print("=" * 50)
    print("Type 'quit' or 'exit' to stop.")
    print("Type 'clear' to reset conversation history.")
    print()

    while True:
        try:
            user_input = input("You: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\nGoodbye!")
            break

        if not user_input:
            continue

        if user_input.lower() in ("quit", "exit"):
            print("Goodbye!")
            break

        if user_input.lower() == "clear":
            conversation_history.clear()
            print("[Conversation history cleared]\n")
            continue

        # Add user message to history
        conversation_history.append({"role": "user", "content": user_input})

        try:
            response = client.messages.create(
                model="claude-sonnet-4-6",
                max_tokens=1024,
                system=system_prompt,
                messages=conversation_history,
            )

            assistant_message = response.content[0].text

            # Add assistant reply to history
            conversation_history.append(
                {"role": "assistant", "content": assistant_message}
            )

            print(f"\nClaude: {assistant_message}\n")

        except anthropic.AuthenticationError:
            print("Error: Invalid API key. Set ANTHROPIC_API_KEY and try again.\n")
        except anthropic.RateLimitError:
            print("Error: Rate limit hit. Please wait a moment and try again.\n")
            conversation_history.pop()  # Remove the unanswered user message
        except anthropic.APIError as e:
            print(f"API error: {e}\n")
            conversation_history.pop()


if __name__ == "__main__":
    run_chatbot()
