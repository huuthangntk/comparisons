# Pros of the `marzban-shop` telegram bot

## List of Pros of `marzban-shop` includes:

```
Payment System: Handles multiple payment providers with clean separation of concerns
Notification System: Automated subscription monitoring and user notifications
Localization System: Flexible multi-language support with fallback handling
Subscription Manager: Comprehensive VPN subscription lifecycle management
Webhook Handler: Secure payment webhook processing with IP verification

Comprehensive Admin Features


Complete admin dashboard
Bulk operation support
Detailed system information
Flexible user management


Robust Template System


Template-based user creation
Multiple application modes (reset/add)
Customizable template parameters
Detailed template information display


Advanced Monitoring


Real-time metrics collection
Historical data storage
Alert system
Comprehensive resource tracking

```

### Multi-Provider Payment System Handler

```
# Location: ./bot/handlers/callbacks.py

from aiogram import Router, F
from aiogram.types import CallbackQuery, LabeledPrice
from aiogram.utils.i18n import gettext as _

class PaymentSystem:
    async def init_yookassa_payment(self, callback, data, user_id, lang_code):
        """Initialize YooKassa payment processing"""
        result = await yookassa.create_payment(
            callback.from_user.id, 
            data, 
            callback.message.chat.id, 
            callback.from_user.language_code
        )
        
        await callback.message.answer(
            _("To be paid - {amount}₽ ⬇️").format(amount=int(result['amount'])),
            reply_markup=get_pay_keyboard(result['url'])
        )

    async def init_crypto_payment(self, callback, data, user_id, lang_code):
        """Initialize Cryptomus payment processing"""
        result = await cryptomus.create_payment(
            callback.from_user.id,
            data,
            callback.message.chat.id,
            callback.from_user.language_code
        )
        
        now = datetime.now()
        expire_date = (now + timedelta(minutes=60)).strftime("%d/%m/%Y, %H:%M")
        
        await callback.message.answer(
            _("To be paid - {amount}$ ⬇️").format(
                amount=result['amount'],
                date=expire_date
            ),
            reply_markup=get_pay_keyboard(result['url'])
        )

    async def init_stars_payment(self, callback, data, good):
        """Initialize Telegram Stars payment"""
        price = good['price']['stars']
        months = good['months']
        prices = [LabeledPrice(label="XTR", amount=price)]
        
        await callback.message.answer_invoice(
            title=_("Subscription for {amount} month").format(amount=months),
            currency="XTR",
            description=_("To be paid - {amount}⭐️ ⬇️").format(amount=int(price)),
            prices=prices,
            provider_token="",
            payload=data,
            reply_markup=get_xtr_pay_keyboard(price)
        )

    async def process_successful_payment(self, message):
        """Handle successful payment completion"""
        good = goods.get(message.successful_payment.invoice_payload)
        user = await get_marzban_profile_db(message.from_user.id)
        await marzban_api.generate_marzban_subscription(user.vpn_id, good)
        await message.answer(
            text=_("Thank you for your purchase ❤️\n️\nSubscription activated."),
            reply_markup=get_main_menu_keyboard(True, message.from_user.language_code)
        )

```

### Automated Notification System

```

# Location: ./bot/tasks/notify_renew_subscription.py

import asyncio
import time
from datetime import datetime, timedelta

class NotificationManager:
    def __init__(self, bot, marzban_api):
        self.bot = bot
        self.marzban_api = marzban_api

    async def notify_expiring_subscriptions(self):
        """Notify users about expiring subscriptions"""
        marzban_users = await self.get_expiring_users()
        if not marzban_users:
            return None
        
        for marzban_user in marzban_users:
            vpn_id = marzban_user['username']
            user = await self.get_user_profile(vpn_id)
            if not user:
                continue

            chat_member = await self.bot.get_chat_member(user.tg_id, user.tg_id)
            if not chat_member:
                continue

            await self.send_expiry_notification(
                user.tg_id,
                chat_member.user.first_name,
                chat_member.user.language_code,
                marzban_user
            )

    async def get_expiring_users(self):
        """Get users with expiring subscriptions"""
        users = await self.marzban_api.get_users()
        if not users:
            return None
            
        now = int(time.time())
        after_tomorrow = now + 60 * 60 * 36
        
        return [
            user for user in users['users'] 
            if user['expire'] and now < user['expire'] < after_tomorrow
        ]

    async def send_expiry_notification(self, user_id, name, lang, subscription):
        """Send expiry notification to user"""
        expire_time = self.get_expiration_time(subscription)
        message = _(
            "Hello, {name} 👋🏻\n\n"
            "Your VPN subscription expires {when}.\n"
            "To renew, please visit the 'Join 🏄🏻‍♂️' section."
        ).format(name=name, when=expire_time)
        
        await self.bot.send_message(user_id, message)

```

### i18n Multi-Language Support

```

# Location: ./bot/utils/lang.py

import gettext
from pathlib import Path
import logging

class LocalizationManager:
    def __init__(self, domain='bot', localedir='locales'):
        self.domain = domain
        self.localedir = Path(__file__).parent.parent / localedir
        self.supported_languages = ['en', 'ru']
        self.default_language = 'en'
        
        # Pre-load translations
        self.translations = {}
        for lang in self.supported_languages:
            try:
                translation = gettext.translation(
                    self.domain,
                    self.localedir,
                    languages=[lang]
                )
                self.translations[lang] = translation
            except Exception as e:
                logging.error(f"Failed to load translations for {lang}: {e}")
                
    def get_string(self, text: str, lang: str = None) -> str:
        """Get localized string for given text and language"""
        if not lang or lang not in self.supported_languages:
            lang = self.default_language
            
        try:
            translation = self.translations.get(lang)
            if translation:
                translation.install()
                return translation.gettext(text)
        except Exception as e:
            logging.error(f"Translation error for {lang}: {e}")
            
        # Fallback to default language
        return self.translations[self.default_language].gettext(text)
        
    def add_language(self, lang_code: str):
        """Add new language support"""
        if lang_code not in self.supported_languages:
            try:
                translation = gettext.translation(
                    self.domain,
                    self.localedir,
                    languages=[lang_code]
                )
                self.translations[lang_code] = translation
                self.supported_languages.append(lang_code)
            except Exception as e:
                logging.error(f"Failed to add language {lang_code}: {e}")

```

### VPN Subscription Management System

```

# Location: ./bot/utils/marzban_api.py

class SubscriptionManager:
    def __init__(self, api_url, access_token):
        self.api_url = api_url
        self.access_token = access_token
        self.protocols = {
            "vmess": [{}],
            "vless": [{"flow": "xtls-rprx-vision"}],
            "trojan": [{}],
            "shadowsocks": [{"method": "chacha20-ietf-poly1305"}]
        }

    async def create_subscription(self, username: str, data_limit: float, expire_days: int):
        """Create new VPN subscription"""
        expiry_time = int(time.time()) + (expire_days * 24 * 60 * 60)
        
        payload = {
            "username": username,
            "proxies": self.protocols,
            "data_limit": data_limit * 1024**3,  # Convert GB to bytes
            "expire": expiry_time,
            "data_limit_reset_strategy": "no_reset",
            "status": "active"
        }
        
        return await self._make_request("POST", "/api/user", payload)

    async def extend_subscription(self, username: str, additional_days: int):
        """Extend existing subscription"""
        user_data = await self.get_subscription(username)
        if not user_data:
            return None
            
        current_expire = user_data.get('expire', int(time.time()))
        new_expire = current_expire + (additional_days * 24 * 60 * 60)
        
        payload = {**user_data, "expire": new_expire}
        return await self._make_request("PUT", f"/api/user/{username}", payload)

    async def get_subscription_status(self, username: str):
        """Get detailed subscription status"""
        data = await self.get_subscription(username)
        if not data:
            return None
            
        return {
            "active": data.get("status") == "active",
            "data_limit": data.get("data_limit", 0) / 1024**3,  # Convert to GB
            "data_used": data.get("used_traffic", 0) / 1024**3,
            "expire_date": datetime.fromtimestamp(data.get("expire", 0)),
            "days_left": (data.get("expire", 0) - int(time.time())) // (24 * 60 * 60)
        }

    async def _make_request(self, method: str, endpoint: str, data: dict = None):
        """Make API request to Marzban panel"""
        headers = {
            "Authorization": f"Bearer {self.access_token}",
            "Content-Type": "application/json"
        }
        
        try:
            async with aiohttp.ClientSession() as session:
                url = f"{self.api_url}{endpoint}"
                async with session.request(method, url, json=data, headers=headers) as resp:
                    if resp.status == 200:
                        return await resp.json()
                    logging.error(f"API Error: {resp.status} - {await resp.text()}")
                    return None
        except Exception as e:
            logging.error(f"Request failed: {str(e)}")
            return None

```

### Webhook Payment Processing System

```
# Location: ./bot/utils/marzban_api.py

class SubscriptionManager:
    def __init__(self, api_url, access_token):
        self.api_url = api_url
        self.access_token = access_token
        self.protocols = {
            "vmess": [{}],
            "vless": [{"flow": "xtls-rprx-vision"}],
            "trojan": [{}],
            "shadowsocks": [{"method": "chacha20-ietf-poly1305"}]
        }

    async def create_subscription(self, username: str, data_limit: float, expire_days: int):
        """Create new VPN subscription"""
        expiry_time = int(time.time()) + (expire_days * 24 * 60 * 60)
        
        payload = {
            "username": username,
            "proxies": self.protocols,
            "data_limit": data_limit * 1024**3,  # Convert GB to bytes
            "expire": expiry_time,
            "data_limit_reset_strategy": "no_reset",
            "status": "active"
        }
        
        return await self._make_request("POST", "/api/user", payload)

    async def extend_subscription(self, username: str, additional_days: int):
        """Extend existing subscription"""
        user_data = await self.get_subscription(username)
        if not user_data:
            return None
            
        current_expire = user_data.get('expire', int(time.time()))
        new_expire = current_expire + (additional_days * 24 * 60 * 60)
        
        payload = {**user_data, "expire": new_expire}
        return await self._make_request("PUT", f"/api/user/{username}", payload)

    async def get_subscription_status(self, username: str):
        """Get detailed subscription status"""
        data = await self.get_subscription(username)
        if not data:
            return None
            
        return {
            "active": data.get("status") == "active",
            "data_limit": data.get("data_limit", 0) / 1024**3,  # Convert to GB
            "data_used": data.get("used_traffic", 0) / 1024**3,
            "expire_date": datetime.fromtimestamp(data.get("expire", 0)),
            "days_left": (data.get("expire", 0) - int(time.time())) // (24 * 60 * 60)
        }

    async def _make_request(self, method: str, endpoint: str, data: dict = None):
        """Make API request to Marzban panel"""
        headers = {
            "Authorization": f"Bearer {self.access_token}",
            "Content-Type": "application/json"
        }
        
        try:
            async with aiohttp.ClientSession() as session:
                url = f"{self.api_url}{endpoint}"
                async with session.request(method, url, json=data, headers=headers) as resp:
                    if resp.status == 200:
                        return await resp.json()
                    logging.error(f"API Error: {resp.status} - {await resp.text()}")
                    return None
        except Exception as e:
            logging.error(f"Request failed: {str(e)}")
            return None

```

### Trial System Implementation

```

# Location: ./bot/handlers/messages.py

from datetime import datetime, timedelta
import time

async def handle_trial_subscription(message: Message):
    """
    Handles trial subscription requests and management.
    Includes validation, creation, and notification.
    """
    result = await had_test_sub(message.from_user.id)
    if result:
        await message.answer(
            _("Your subscription is available in the \"My subscription 👤\" section."),
            reply_markup=get_main_menu_keyboard(True))
        return

    # Create trial subscription
    user = await get_marzban_profile_db(message.from_user.id)
    result = await generate_test_subscription(user.vpn_id)
    await update_test_subscription_state(message.from_user.id)
    
    # Notify user
    await send_trial_welcome_message(message, user)

def get_test_subscription(hours: int, additional=False) -> int:
    """Calculate trial subscription expiry timestamp"""
    return (0 if additional else int(time.time())) + 60 * 60 * hours

async def validate_trial_eligibility(user_id: int) -> bool:
    """
    Validates if user is eligible for trial subscription
    Checks previous usage and time restrictions
    """
    async with GetDB() as db:
        # Check previous trial usage
        previous_trial = await db.get_user_trial_history(user_id)
        if previous_trial:
            return False
            
        # Check time-based restrictions
        last_account = await db.get_user_last_account(user_id)
        if last_account and (datetime.now() - last_account.created_at) < timedelta(days=30):
            return False
            
        return True

# Trial status tracking
async def update_test_subscription_state(user_id: int):
    """Updates user's trial subscription state"""
    async with GetDB() as db:
        sql_q = update(VPNUsers).where(
            VPNUsers.tg_id == user_id
        ).values(test=True)
        await db.execute(sql_q)
        await db.commit()

```

### Flexible Configuration System

```

# Location: ./bot/glv.py and .env.example

import os
from typing import Dict, Any

class ConfigManager:
    """
    Manages bot configuration with support for multiple environments
    and dynamic settings management.
    """
    def __init__(self):
        self.config: Dict[str, Any] = {
            'BOT_TOKEN': os.environ.get('BOT_TOKEN'),
            'SHOP_NAME': os.environ.get('SHOP_NAME'),
            'PROTOCOLS': os.environ.get('PROTOCOLS', 'vless').split(),
            'TEST_PERIOD': os.environ.get('TEST_PERIOD', False) == 'true',
            'PERIOD_LIMIT': int(os.environ.get('PERIOD_LIMIT', 72)),
            
            # Payment System Config
            'YOOKASSA_TOKEN': os.environ.get('YOOKASSA_TOKEN'),
            'YOOKASSA_SHOPID': os.environ.get('YOOKASSA_SHOPID'),
            'CRYPTO_TOKEN': os.environ.get('CRYPTO_TOKEN'),
            'MERCHANT_UUID': os.environ.get('MERCHANT_UUID'),
            'STARS_PAYMENT_ENABLED': os.environ.get('STARS_PAYMENT_ENABLED', False) == 'true',
            
            # Notification Settings
            'RENEW_NOTIFICATION_TIME': str(os.environ.get('RENEW_NOTIFICATION_TIME')),
            'EXPIRED_NOTIFICATION_TIME': str(os.environ.get('EXPIRED_NOTIFICATION_TIME')),
            'TG_INFO_CHANEL': os.environ.get('TG_INFO_CHANEL'),
            
            # Panel Configuration
            'PANEL_HOST': os.environ.get('PANEL_HOST'),
            'PANEL_GLOBAL': os.environ.get('PANEL_GLOBAL'),
            'PANEL_USER': os.environ.get('PANEL_USER'),
            'PANEL_PASS': os.environ.get('PANEL_PASS'),
        }

    def get(self, key: str, default: Any = None) -> Any:
        """Get configuration value with optional default"""
        return self.config.get(key, default)

    def update(self, key: str, value: Any) -> None:
        """Update configuration value"""
        self.config[key] = value
        
    def get_payment_configs(self) -> Dict[str, bool]:
        """Get status of available payment methods"""
        return {
            'yookassa': bool(self.config['YOOKASSA_SHOPID'] and self.config['YOOKASSA_TOKEN']),
            'crypto': bool(self.config['MERCHANT_UUID'] and self.config['CRYPTO_TOKEN']),
            'stars': self.config['STARS_PAYMENT_ENABLED']
        }

    def validate_required_configs(self) -> bool:
        """Validate all required configuration values are present"""
        required_keys = ['BOT_TOKEN', 'PANEL_HOST', 'PANEL_USER', 'PANEL_PASS']
        return all(self.config.get(key) for key in required_keys)

# Example .env configuration
"""
BOT_TOKEN=your_bot_token
SHOP_NAME=VPN Shop
PROTOCOLS=vless,vmess,trojan
TEST_PERIOD=true
PERIOD_LIMIT=72
RENEW_NOTIFICATION_TIME=16:00
EXPIRED_NOTIFICATION_TIME=16:05
TG_INFO_CHANEL=@channel_name

# Payment Configuration
YOOKASSA_TOKEN=your_token
YOOKASSA_SHOPID=your_shop_id
CRYPTO_TOKEN=your_token
MERCHANT_UUID=your_uuid
STARS_PAYMENT_ENABLED=false

# Panel Configuration  
PANEL_HOST=http://localhost:8000
PANEL_GLOBAL=https://your-domain.com
PANEL_USER=admin
PANEL_PASS=password
"""

```

### User-Friendly Interface Implementation

```

# Location: ./bot/keyboards/ and handlers/

from aiogram.types import ReplyKeyboardMarkup, InlineKeyboardMarkup, KeyboardButton
from aiogram.utils.i18n import gettext as _

class UserInterface:
    """
    Manages the bot's user interface components with support for:
    - Multi-language menus
    - Dynamic button generation
    - Context-aware navigation
    """
    
    @staticmethod
    def get_main_menu(trial_expired: bool, lang=None) -> ReplyKeyboardMarkup:
        """Creates main menu with dynamic buttons based on user state"""
        buttons = [
            [KeyboardButton(text=_("Join 🏄🏻‍♂️"))],
            [KeyboardButton(text=_("Support ❤️"))]
        ]
        
        status_buttons = [
            KeyboardButton(text=_("Frequent questions ℹ️"))
        ]
        
        if trial_expired:
            status_buttons.insert(0, KeyboardButton(text=_("My subscription 👤")))
        
        buttons.insert(1, status_buttons)
        
        # Add trial button if available
        if not trial_expired:
            buttons.insert(0, [KeyboardButton(text=_("5 days free 🆓"))])
            
        return ReplyKeyboardMarkup(
            keyboard=buttons,
            resize_keyboard=True,
            is_persistent=True
        )

    @staticmethod
    def get_subscription_info(user, lang=None) -> str:
        """Formats subscription information in a user-friendly way"""
        status_icons = {
            'active': '✅',
            'expired': '🕰',
            'limited': '📵',
            'disabled': '❌'
        }
        
        return f"""\
┌─{status_icons[user.status]} Status: {user.status.title()}
│          └─Username: {user.username}
│
├─🔋 Data limit: {format_data_limit(user.data_limit)}
│          └─Data Used: {format_data_usage(user.used_traffic)}
│
└─📅 Expiry Date: {format_expiry_date(user.expire)}
"""

    @staticmethod
    def get_payment_methods(good: dict) -> InlineKeyboardMarkup:
        """Creates payment method selection keyboard"""
        keyboard = InlineKeyboardMarkup()
        
        if config.get_payment_configs()['yookassa']:
            keyboard.add(Button(
                text=_("YooKassa - ₽"),
                callback_data=f"pay_kassa_{good['callback']}"
            ))
            
        if config.get_payment_configs()['crypto']:
            keyboard.add(Button(
                text="Cryptomus - $",
                callback_data=f"pay_crypto_{good['callback']}"
            ))
            
        if config.get_payment_configs()['stars']:
            keyboard.add(Button(
                text="Telegram Stars - ⭐️",
                callback_data=f"pay_stars_{good['callback']}"
            ))
            
        return keyboard

    @staticmethod
    def handle_navigation(message, prev_state: str = None):
        """Handles menu navigation with state management"""
        states = {
            'main': get_main_menu,
            'subscription': show_subscription,
            'payment': show_payment_options,
            'support': show_support,
            'trial': handle_trial
        }
        
        handler = states.get(message.text.lower(), states['main'])
        return handler(message)

```