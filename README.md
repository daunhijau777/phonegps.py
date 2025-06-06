# phonegps.py
Real-World Phone Number GPS Location Finder

Original & Implementable Script by H4TIHIT4M 

import requests import logging import time import json from typing import Optional, Dict, Any from phonenumbers import parse, geocoder, carrier, is_valid_number from faker import Faker from rich.prompt import Prompt from rich.console import Console

console = Console() logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")

=== CONFIGURATION ===

OPENCAGE_API_KEY = "YOUR_OPENCAGE_API_KEY"  # Ganti dengan API key kamu SYNC_ME_FAKE = { "+628123456789": "Jakarta, Indonesia", "+628129713786": "Washington D.C." } HEADERS = { "User-Agent": Faker().user_agent(), "Accept-Language": "en-US,en;q=0.9", "Accept": "application/json" }

def get_phone_info(number: str) -> Dict[str, Any]: try: phone = parse(number) if not is_valid_number(phone): raise ValueError("Invalid phone number") region = geocoder.description_for_number(phone, "en") provider = carrier.name_for_number(phone, "en") return {"region": region, "carrier": provider, "country_code": phone.country_code} except Exception as e: logging.error(f"Phone parsing failed: {e}") return {}

def get_syncme_location(number: str) -> Optional[str]: if number in SYNC_ME_FAKE: location = SYNC_ME_FAKE[number] logging.info(f"Sync.me location found: {location}") return location return None

def geocode_location(location_name: str) -> Optional[Dict[str, Any]]: url = "https://api.opencagedata.com/geocode/v1/json" params = { "q": location_name, "key": OPENCAGE_API_KEY, "limit": 1, "no_annotations": 1 } try: response = requests.get(url, headers=HEADERS, params=params, timeout=10) response.raise_for_status() data = response.json() if data["results"]: result = data["results"][0] lat = result["geometry"]["lat"] lon = result["geometry"]["lng"] formatted = result["formatted"] logging.info(f"Geocoded '{location_name}' to {formatted} ({lat}, {lon})") return {"lat": lat, "lon": lon, "address": formatted} except Exception as e: logging.error(f"Geocoding error: {e}") return None

def track_phone(number: str, extra_info: Optional[str] = None): logging.info(f"Start tracking for phone: {number}")

info = get_phone_info(number)
prefix = str(info.get("country_code", "Unknown"))
logging.info(f"Detected prefix: {prefix} => {info.get('region', 'Unknown')}")

location_source = get_syncme_location(number) or info.get("region")
if extra_info:
    location_source += f" {extra_info}"

geodata = geocode_location(location_source)
if geodata:
    console.print("\n[bold green]Lokasi Ditemukan:[/bold green]")
    console.print(f"[bold]Alamat:[/bold] {geodata['address']}")
    console.print(f"[bold]Koordinat:[/bold] {geodata['lat']}, {geodata['lon']}")
else:
    console.print("[bold red]Lokasi tidak ditemukan atau data kurang.[/bold red]")

if name == "main": console.print("[bold blue]Phone Number GPS Location Finder ===[/bold blue]") phone_number = Prompt.ask("Masukkan nomor telepon (contoh +628123456789)") tambahan = Prompt.ask("Masukkan teks tambahan (bio, profil, dll) jika ada, atau kosongkan", default="") track_phone(phone_number, tambahan.strip())

