# Pros of the `greed` telegram bot

## Advanced Transaction Management System

```
import datetime
from decimal import Decimal
from typing import Optional, Union

class Price:
    """A robust price handling system with currency formatting and mathematical operations."""
    
    def __init__(self, value: Union[int, float, str, "Price"], currency_exp: int = 2):
        self.currency_exp = currency_exp
        
        if isinstance(value, int):
            self.value = int(value)
        elif isinstance(value, float):
            self.value = int(value * (10 ** self.currency_exp))
        elif isinstance(value, str):
            self.value = int(float(value.replace(",", ".")) * (10 ** self.currency_exp))
        elif isinstance(value, Price):
            self.value = value.value
            
    def __str__(self):
        return self.format_currency(
            symbol=self.currency_symbol,
            value="{0:.2f}".format(self.value / (10 ** self.currency_exp))
        )

    def __int__(self):
        return self.value
        
    def __add__(self, other):
        return Price(self.value + Price(other).value)
        
    def __sub__(self, other):
        return Price(self.value - Price(other).value)

class Transaction:
    """Enhanced transaction handling with support for refunds and detailed logging."""
    
    def __init__(self, user_id: int, amount: Price, provider: str, notes: str = None):
        self.user_id = user_id
        self.amount = amount
        self.provider = provider
        self.notes = notes
        self.timestamp = datetime.datetime.now()
        self.refunded = False
        self.refund_reason = None
        self.payment_details = {}

    def refund(self, reason: str) -> bool:
        """Process a refund with proper logging and state management."""
        if self.refunded:
            return False
            
        self.refunded = True
        self.refund_reason = reason
        self.refund_timestamp = datetime.datetime.now()
        return True

    def add_payment_details(self, **kwargs):
        """Store additional payment provider specific details."""
        self.payment_details.update(kwargs)

    def to_dict(self) -> dict:
        """Convert transaction to dictionary for storage/serialization."""
        return {
            "id": self.id,
            "user_id": self.user_id,
            "amount": self.amount.value,
            "provider": self.provider,
            "notes": self.notes,
            "timestamp": self.timestamp.isoformat(),
            "refunded": self.refunded,
            "refund_reason": self.refund_reason,
            "refund_timestamp": self.refund_timestamp.isoformat() if self.refunded else None,
            "payment_details": self.payment_details
        }

class TransactionManager:
    """Manages transaction processing, refunds, and history."""
    
    def create_transaction(self, user_id: int, amount: Price, provider: str, notes: str = None) -> Transaction:
        """Create and record a new transaction."""
        transaction = Transaction(user_id, amount, provider, notes)
        self._save_transaction(transaction)
        self._update_user_balance(user_id, amount)
        return transaction
        
    def process_refund(self, transaction_id: int, reason: str) -> bool:
        """Process a refund for a specific transaction."""
        transaction = self._get_transaction(transaction_id)
        if not transaction or transaction.refunded:
            return False
            
        success = transaction.refund(reason)
        if success:
            self._update_user_balance(transaction.user_id, -transaction.amount)
            self._save_transaction(transaction)
        return success

    def get_user_transactions(self, user_id: int, start_date: datetime = None, 
                            end_date: datetime = None) -> list[Transaction]:
        """Retrieve filtered transaction history for a user."""
        transactions = self._load_user_transactions(user_id)
        
        if start_date:
            transactions = [t for t in transactions if t.timestamp >= start_date]
        if end_date:
            transactions = [t for t in transactions if t.timestamp <= end_date]
            
        return transactions
```

## Advanced Conversation Flow Management

```
from typing import Optional, Callable, Any
from enum import Enum, auto

class ConversationState(Enum):
    STARTING = auto()
    AWAITING_INPUT = auto()
    PROCESSING = auto()
    CANCELLED = auto()
    COMPLETED = auto()
    TIMEOUT = auto()

class ConversationHandler:
    """Advanced conversation flow management with state tracking and timeout handling."""
    
    def __init__(self, timeout: int = 300):
        self.state = ConversationState.STARTING
        self.timeout = timeout
        self.last_activity = datetime.datetime.now()
        self.context = {}
        self._handlers = {}
        
    async def handle_message(self, message: dict) -> Optional[str]:
        """Process incoming messages based on current state and registered handlers."""
        if self._is_timeout():
            self.state = ConversationState.TIMEOUT
            return self._handle_timeout()
            
        self.last_activity = datetime.datetime.now()
        
        handler = self._handlers.get(self.state)
        if not handler:
            return None
            
        try:
            result = await handler(message, self.context)
            return result
        except Exception as e:
            return self._handle_error(e)
            
    def register_handler(self, state: ConversationState, 
                        handler: Callable[[dict, dict], Any]):
        """Register a handler for a specific conversation state."""
        self._handlers[state] = handler
        
    def set_state(self, new_state: ConversationState):
        """Update conversation state with validation."""
        if new_state not in ConversationState:
            raise ValueError(f"Invalid state: {new_state}")
        self.state = new_state
        
    def add_context(self, key: str, value: Any):
        """Store data in conversation context."""
        self.context[key] = value
        
    def _is_timeout(self) -> bool:
        """Check if conversation has timed out."""
        elapsed = (datetime.datetime.now() - self.last_activity).total_seconds()
        return elapsed > self.timeout
        
    def _handle_timeout(self) -> str:
        """Handle conversation timeout."""
        return "Conversation timed out. Please start again."
        
    def _handle_error(self, error: Exception) -> str:
        """Handle conversation errors."""
        # Log error details
        return "An error occurred. Please try again."

class ConversationManager:
    """Manages multiple concurrent conversations."""
    
    def __init__(self):
        self.conversations = {}
        
    def get_conversation(self, user_id: int) -> ConversationHandler:
        """Get or create conversation handler for user."""
        if user_id not in self.conversations:
            self.conversations[user_id] = ConversationHandler()
        return self.conversations[user_id]
        
    def end_conversation(self, user_id: int):
        """Clean up completed conversation."""
        if user_id in self.conversations:
            del self.conversations[user_id]
            
    async def process_message(self, user_id: int, message: dict) -> Optional[str]:
        """Process message in correct conversation context."""
        conversation = self.get_conversation(user_id)
        response = await conversation.handle_message(message)
        
        if conversation.state in [ConversationState.COMPLETED, 
                                ConversationState.CANCELLED,
                                ConversationState.TIMEOUT]:
            self.end_conversation(user_id)
            
        return response
```

## Role-Based Admin Management System

```
from enum import Flag, auto
from typing import List, Optional

class AdminPermission(Flag):
    """Granular permission management using flag-based system."""
    NONE = 0
    VIEW_PRODUCTS = auto()
    EDIT_PRODUCTS = auto()
    VIEW_ORDERS = auto()
    PROCESS_ORDERS = auto()
    VIEW_TRANSACTIONS = auto()
    CREATE_TRANSACTIONS = auto()
    MANAGE_USERS = auto()
    MANAGE_ADMINS = auto()
    
    # Permission combinations
    BASIC_ADMIN = VIEW_PRODUCTS | VIEW_ORDERS | VIEW_TRANSACTIONS
    FULL_ADMIN = ~NONE
    
class AdminRole:
    """Role definition with permission sets."""
    
    def __init__(self, name: str, permissions: AdminPermission, 
                 display_on_help: bool = False):
        self.name = name
        self.permissions = permissions
        self.display_on_help = display_on_help
        
    def has_permission(self, permission: AdminPermission) -> bool:
        """Check if role has specific permission."""
        return bool(self.permissions & permission)
        
    def grant_permission(self, permission: AdminPermission):
        """Add permission to role."""
        self.permissions |= permission
        
    def revoke_permission(self, permission: AdminPermission):
        """Remove permission from role."""
        self.permissions &= ~permission

class Admin:
    """Admin user with role-based permissions."""
    
    def __init__(self, user_id: int, role: AdminRole):
        self.user_id = user_id
        self.role = role
        self.is_active = True
        self.last_action = None
        
    def can(self, permission: AdminPermission) -> bool:
        """Check if admin has specific permission."""
        return self.is_active and self.role.has_permission(permission)

class AdminManager:
    """Manages admin users and roles."""
    
    def __init__(self):
        self.admins: dict[int, Admin] = {}
        self.roles: dict[str, AdminRole] = {
            "basic": AdminRole("Basic Admin", AdminPermission.BASIC_ADMIN),
            "full": AdminRole("Full Admin", AdminPermission.FULL_ADMIN, True)
        }
        
    def add_admin(self, user_id: int, role_name: str) -> Optional[Admin]:
        """Create new admin with specified role."""
        if role_name not in self.roles:
            return None
            
        admin = Admin(user_id, self.roles[role_name])
        self.admins[user_id] = admin
        return admin
        
    def remove_admin(self, user_id: int):
        """Remove admin status from user."""
        if user_id in self.admins:
            del self.admins[user_id]
            
    def get_admins_by_permission(self, permission: AdminPermission) -> List[Admin]:
        """Get all admins with specific permission."""
        return [admin for admin in self.admins.values() 
                if admin.can(permission)]
                
    def create_role(self, name: str, permissions: AdminPermission, 
                   display_on_help: bool = False) -> AdminRole:
        """Create custom admin role."""
        role = AdminRole(name, permissions, display_on_help)
        self.roles[name] = role
        return role
```

## Flexible Localization System

```
from typing import Dict, Any, Optional
import json
import os

class LocalizationManager:
    """Advanced localization system with fallback support and dynamic string formatting."""
    
    def __init__(self, default_language: str = "en", 
                 fallback_language: str = "en"):
        self.default_language = default_language
        self.fallback_language = fallback_language
        self.strings: Dict[str, Dict[str, str]] = {}
        self.replacements: Dict[str, Dict[str, Any]] = {}
        
    def load_language(self, language_code: str, file_path: str):
        """Load language strings from JSON file."""
        try:
            with open(file_path, 'r', encoding='utf-8') as f:
                self.strings[language_code] = json.load(f)
        except FileNotFoundError:
            print(f"Language file not found: {file_path}")
        except json.JSONDecodeError:
            print(f"Invalid JSON in language file: {file_path}")
            
    def load_languages_from_directory(self, directory: str):
        """Load all language files from a directory."""
        for filename in os.listdir(directory):
            if filename.endswith('.json'):
                language_code = filename[:-5]
                self.load_language(language_code, 
                                 os.path.join(directory, filename))
                
    def add_replacements(self, language_code: str, 
                        replacements: Dict[str, Any]):
        """Add dynamic replacement values for a language."""
        if language_code not in self.replacements:
            self.replacements[language_code] = {}
        self.replacements[language_code].update(replacements)
        
    def get(self, key: str, language_code: str = None, **kwargs) -> str:
        """Get localized string with fallback support and formatting."""
        if not language_code:
            language_code = self.default_language
            
        # Try requested language
        if language_code in self.strings and key in self.strings[language_code]:
            string = self.strings[language_code][key]
        # Try fallback language
        elif key in self.strings[self.fallback_language]:
            string = self.strings[self.fallback_language][key]
        # Key not found
        else:
            return f"Missing string: {key}"
            
        # Apply language-specific replacements
        if language_code in self.replacements:
            kwargs.update(self.replacements[language_code])
            
        try:
            return string.format(**kwargs)
        except KeyError as e:
            return f"Missing replacement in string {key}: {e}"
            
    def get_supported_languages(self) -> list[str]:
        """Get list of supported language codes."""
        return list(self.strings.keys())
        
    def validate_strings(self) -> list[str]:
        """Validate all language files have consistent keys."""
        errors = []
        reference_keys = set(self.strings[self.fallback_language].keys())
        
        for lang, strings in self.strings.items():
            if lang == self.fallback_language:
                continue
                
            lang_keys = set(strings.keys())
            
            # Check for missing keys
            missing = reference_keys - lang_keys
            if missing:
                errors.append(f"Language {lang} missing keys: {missing}")
                
            # Check for extra keys
            extra = lang_keys - reference_keys
            if extra:
                errors.append(f"Language {lang} has extra keys: {extra}")
                
        return errors
```