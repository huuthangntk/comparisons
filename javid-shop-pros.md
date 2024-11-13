# Pros of the `javid-shop` telegram bot

## Marzban API Facade Pattern

```
class MarzbanApiFacade:
    """
    A clean facade pattern for Marzban API interactions.
    Key improvements:
    - Centralized API communication
    - Consistent error handling
    - Clean separation from business logic
    """
    
    @staticmethod
    def get_access_token():
        try:
            url = f'{os.getenv("MARZBAN_API_HOST")}/api/admin/token'
            data = {
                'username': os.getenv("MARZBAN_ADMIN_USERNAME"),
                'password': os.getenv("MARZBAN_ADMIN_PASSWORD"),
                'grant_type': 'password'
            }
            response = requests.post(url, data=data)
            return response.json()['access_token']
        except Exception:
            logger.error('Exception -> get_access_token -> ', exc_info=True)

    @staticmethod
    def create_user(telegram_user_id, access_token):
        try:
            url = f'{os.getenv("MARZBAN_API_HOST")}/api/user'
            data = {
                "username": telegram_user_id,
                "proxies": {
                    "vless": {
                        "flow": "xtls-rprx-vision"
                    }
                },
                "inbounds": {
                    "vless": [
                        "VLESS TCP REALITY"
                    ]
                },
                "expire": 0,
                "data_limit": 0,
                "data_limit_reset_strategy": "no_reset",
                "status": "active"
            }
            headers = {
                'Authorization': f'Bearer {access_token}'
            }
            response = requests.post(url, headers=headers, json=data)
            return [response.json(), response.status_code]
        except Exception:
            logger.error('Exception -> create_user -> ', exc_info=True)
```

## Marzban Service Layer Pattern

```

class MarzbanService:
    """
    Service layer that provides high-level business operations.
    Key features:
    - Abstracts API complexity
    - Handles token management
    - Combines multiple API calls into single operations
    """

    @staticmethod
    def create_marzaban_user(telegram_user_id):
        """Creates a new user and handles all necessary API interactions"""
        access_token = MarzbanService.access_token()
        api_response, status_code = MarzbanApiFacade.create_user(telegram_user_id, access_token)
        return [api_response, status_code, access_token]

    @staticmethod
    def get_marzaban_user(telegram_user_id, access_token):
        """Retrieves user data with error handling"""
        api_response, status_code = MarzbanApiFacade.get_user(telegram_user_id, access_token)
        return [api_response, status_code]

    @staticmethod
    def refresh_all_users():
        """Example of a high-level operation combining multiple API calls"""
        access_token = MarzbanService.access_token()
        # Implementation for refreshing all users
        return access_token

```

## User Repository Pattern

```
class UserRepository:
    """
    Repository pattern for database operations.
    Features:
    - Transaction management
    - Error handling
    - Clean database access abstraction
    """
    
    @staticmethod
    def get_user(telegram_user_id):
        with Session() as session:
            with session.begin():
                try:
                    logger.info('get_user -> grabbing user\'s data')
                    user = session.query(User).filter_by(telegram_user_id=telegram_user_id).first()
                    configurations = None if user == None else user.configurations
                    session.close()
                except Exception:
                    logger.error("Exception -> get_user: ", exc_info=True)
                    session.rollback()
        return [user, configurations]

    @staticmethod
    def refresh_configs(access_token):
        """Example of complex database operation with error handling"""
        session = Session()
        users = session.query(User).all()
        for user in users:
            try:
                # Get latest user data from Marzban
                user_marzban_data, status_code = MarzbanApiFacade.get_user(
                    user.telegram_user_id, 
                    access_token
                )
                if status_code == 200:
                    # Update configurations
                    new_configs = [
                        Configurations(user.telegram_user_id, link) 
                        for link in user_marzban_data['links']
                    ]
                    existing_configs = session.query(Configurations).filter_by(
                        telegram_user_id=user.telegram_user_id
                    ).all()
                    
                    # Handle updates in transaction
                    with session.begin():
                        for config in existing_configs:
                            session.delete(config)
                        session.bulk_save_objects(new_configs)
                        
            except Exception:
                logger.error("Exception -> refresh_configs: ", exc_info=True)
                session.rollback()

```

## Structured Message Content System

```

{
    "unexpected_error": "Something went wrong, please try again later!",
    "welcome": "Hello, Welcome to Marzban, Click the button below to create your configurations!",
    "welcome_back": "Welcome Back!, Use the buttons below to access available configurations and usage manuals",
    "refresh_started": "Refresh in progress...",
    "refresh_done": "Refresh completed successfully!",
    "configs_panel": "Find below the available Configs!",
    "default_fallback": "Unknown Message!",
    "created_configs": "Configurations created successfully, Use the buttons below to access available ones",
    "manuals": "Please read this <a href=\"{link}\">manual</a> carefully",
    "link_available": "Link is ready!{breakpoint}`{link}`",
    "link_unavailable": "No Link available for {location}",
    "configs_exist": "Configs already exist, use the buttons below to access your configs!",
    "no_configs": "No Configs Available, Please click the button below to create configs"
}

```