"""
Customer Service Chatbot
========================
A rule-based NLP chatbot using AIML (Artificial Intelligence Markup Language).
Uses pattern matching and template responses to answer common customer queries.

Author: [Your Name]
Date: 2024
"""

import aiml
import os
import re
import sys
import datetime


class CustomerServiceBot:
    """
    A customer service chatbot powered by AIML pattern matching.
    Supports intent detection, session management, and conversation logging.
    """

    def __init__(self, aiml_path: str = None):
        """
        Initialize the chatbot and load AIML knowledge base.

        Args:
            aiml_path (str): Path to the AIML file or directory.
        """
        self.kernel = aiml.Kernel()
        self.kernel.verbose(False)
        self.session_id = "user_session_001"
        self.conversation_log = []
        self.intent_stats = {}

        if aiml_path is None:
            base_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
            aiml_path = os.path.join(base_dir, "data", "customer_service.aiml")

        self._load_aiml(aiml_path)
        self._setup_session()

    def _load_aiml(self, aiml_path: str):
        """Load the AIML knowledge base file."""
        if not os.path.exists(aiml_path):
            raise FileNotFoundError(f"AIML file not found: {aiml_path}")

        print(f"Loading AIML knowledge base from: {aiml_path}")
        self.kernel.learn(aiml_path)
        print("Knowledge base loaded successfully!\n")

    def _setup_session(self):
        """Initialize a user session with default predicates."""
        self.kernel.setPredicate("name", "Customer", self.session_id)
        self.kernel.setPredicate("topic", "general", self.session_id)

    def preprocess(self, user_input: str) -> str:
        """
        Preprocess user input for better pattern matching.
        - Convert to uppercase (AIML patterns are uppercase)
        - Remove extra whitespace
        - Expand common abbreviations

        Args:
            user_input (str): Raw user input.
        Returns:
            str: Preprocessed input string.
        """
        text = user_input.strip().upper()

        # Expand abbreviations
        abbreviations = {
            "PLS": "PLEASE",
            "PLZ": "PLEASE",
            "CAN'T": "CANNOT",
            "WON'T": "WILL NOT",
            "DIDN'T": "DID NOT",
            "DON'T": "DO NOT",
            "I'M": "I AM",
            "I'VE": "I HAVE",
            "IT'S": "IT IS",
            "WHAT'S": "WHAT IS",
            "WHERE'S": "WHERE IS",
            "HOW'S": "HOW IS",
        }
        for abbr, full in abbreviations.items():
            text = re.sub(r'\b' + abbr + r'\b', full, text)

        # Remove punctuation except essential ones
        text = re.sub(r'[^\w\s]', '', text)
        text = re.sub(r'\s+', ' ', text).strip()

        return text

    def detect_intent(self, user_input: str) -> str:
        """
        Detect the high-level intent category of the user's message.

        Args:
            user_input (str): Preprocessed user input.
        Returns:
            str: Detected intent label.
        """
        text = user_input.lower()
        intent_keywords = {
            "order_tracking":   ["order", "track", "tracking", "where is", "shipped", "delivery"],
            "returns_refunds":  ["return", "refund", "exchange", "send back", "money back"],
            "shipping":         ["shipping", "ship", "delivery", "free shipping", "international"],
            "payment":          ["payment", "pay", "credit card", "paypal", "billing", "charge"],
            "account":          ["account", "password", "login", "sign in", "register", "delete"],
            "product":          ["product", "stock", "available", "warranty", "item"],
            "subscription":     ["subscription", "cancel", "plan", "membership", "renew"],
            "discount":         ["discount", "promo", "coupon", "deal", "offer", "sale"],
            "complaint":        ["complaint", "unhappy", "problem", "issue", "wrong", "broken"],
            "greeting":         ["hello", "hi", "hey", "morning", "afternoon", "evening"],
            "farewell":         ["bye", "goodbye", "thank", "thanks", "see you"],
            "escalation":       ["human", "agent", "person", "representative", "support"],
        }

        scores = {intent: 0 for intent in intent_keywords}
        for intent, keywords in intent_keywords.items():
            for kw in keywords:
                if kw in text:
                    scores[intent] += 1

        best_intent = max(scores, key=scores.get)
        if scores[best_intent] == 0:
            return "unknown"

        # Track intent stats
        self.intent_stats[best_intent] = self.intent_stats.get(best_intent, 0) + 1
        return best_intent

    def get_response(self, user_input: str) -> dict:
        """
        Generate a response for the given user input.

        Args:
            user_input (str): Raw user message.
        Returns:
            dict: Response containing message, intent, and timestamp.
        """
        if not user_input.strip():
            return {
                "response": "Please type a message so I can help you!",
                "intent": "empty",
                "timestamp": datetime.datetime.now().isoformat()
            }

        processed = self.preprocess(user_input)
        intent = self.detect_intent(user_input)
        response = self.kernel.respond(processed, self.session_id)

        if not response or response.strip() == "":
            response = ("I'm sorry, I didn't understand that. I can help with orders, "
                       "returns, shipping, payments, and account issues. "
                       "Type 'CONTACT SUPPORT' to speak with a human agent.")

        log_entry = {
            "timestamp": datetime.datetime.now().isoformat(),
            "user": user_input,
            "processed": processed,
            "intent": intent,
            "bot": response
        }
        self.conversation_log.append(log_entry)

        return {
            "response": response,
            "intent": intent,
            "timestamp": log_entry["timestamp"]
        }

    def save_conversation_log(self, filepath: str = "conversation_log.txt"):
        """
        Save the full conversation log to a text file.

        Args:
            filepath (str): Output file path.
        """
        with open(filepath, "w", encoding="utf-8") as f:
            f.write("=" * 60 + "\n")
            f.write("CUSTOMER SERVICE CHATBOT - CONVERSATION LOG\n")
            f.write("=" * 60 + "\n\n")

            for entry in self.conversation_log:
                f.write(f"[{entry['timestamp']}]\n")
                f.write(f"User    : {entry['user']}\n")
                f.write(f"Intent  : {entry['intent']}\n")
                f.write(f"Bot     : {entry['bot']}\n")
                f.write("-" * 40 + "\n")

            f.write("\n\nINTENT STATISTICS:\n")
            for intent, count in sorted(self.intent_stats.items(), key=lambda x: -x[1]):
                f.write(f"  {intent}: {count} times\n")

        print(f"\nConversation log saved to: {filepath}")

    def get_stats(self) -> dict:
        """Return chatbot session statistics."""
        return {
            "total_messages": len(self.conversation_log),
            "intent_breakdown": self.intent_stats,
            "session_id": self.session_id
        }


def run_cli_chatbot():
    """Run the chatbot in interactive CLI mode."""
    print("=" * 60)
    print("  CUSTOMER SERVICE CHATBOT")
    print("  Powered by AIML Pattern Matching")
    print("=" * 60)
    print("Type your question below. Type 'quit' or 'exit' to end.")
    print("Type 'stats' to see conversation statistics.\n")

    try:
        bot = CustomerServiceBot()
    except FileNotFoundError as e:
        print(f"Error: {e}")
        sys.exit(1)

    # Greet the user
    opening = bot.get_response("hello")
    print(f"Bot: {opening['response']}\n")

    while True:
        try:
            user_input = input("You: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\n\nSession ended by user.")
            break

        if not user_input:
            continue

        if user_input.lower() in ["quit", "exit", "q"]:
            farewell = bot.get_response("bye")
            print(f"Bot: {farewell['response']}")
            break

        if user_input.lower() == "stats":
            stats = bot.get_stats()
            print(f"\n--- Session Stats ---")
            print(f"Total messages: {stats['total_messages']}")
            print(f"Intent breakdown: {stats['intent_breakdown']}")
            print("--------------------\n")
            continue

        result = bot.get_response(user_input)
        print(f"Bot [{result['intent']}]: {result['response']}\n")

    # Offer to save log
    save = input("\nSave conversation log? (y/n): ").strip().lower()
    if save == "y":
        bot.save_conversation_log("conversation_log.txt")

    print("\nThank you for using the Customer Service Chatbot!")


if __name__ == "__main__":
    run_cli_chatbot()
