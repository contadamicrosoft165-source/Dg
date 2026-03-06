import sys
import re
from typing import List, Dict, Tuple

class ESPHighlighter:
    """Universal ESP highlighting tool for security tools"""
    
    # Common ESP patterns to highlight
    ESP_PATTERNS = {
        "email": r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
        "ip": r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b',
        "url": r'https?://(?:[-\w.]|(?:%[\da-fA-F]{2}))+',
        "password": r'pass(?:word)?[:=]\s*["\']?(.*?)(?=["\'])',
        "key": r'(?:api|access|secret|private)_key\s*[:=]\s*["\']?(.*?)(?=["\'])',
        "token": r'(?:auth|jwt|bearer)\s*[:=]\s*["\']?(.*?)(?=["\'])'
    }
    
    def __init__(self, output_format="ansi"):
        self.output_format = output_format
        
    def highlight(self, text: str) -> str:
        """Apply highlighting to text based on ESP patterns"""
        highlighted = text
        matches = []
        
        for name, pattern in self.ESP_PATTERNS.items():
            for match in re.finditer(pattern, text):
                matches.append((match.start(), match.end(), name))
                
        # Sort matches by position
        matches.sort()
        
        # Apply highlighting
        result = []
        last_pos = 0
        
        for start, end, pattern_type in matches:
            result.append(text[last_pos:start])
            
            if self.output_format == "ansi":
                result.append(f"\033[31m{text[start:end]}\033[0m")
            elif self.output_format == "html":
                result.append(f"<span class='{pattern_type}'>{text[start:end]}</span>")
            else:
                result.append(f"[{pattern_type}]{text[start:end]}[/]")
                
            last_pos = end
            
        result.append(text[last_pos:])
        return "".join(result)

def main():
    """Main function for CLI usage"""
    if len(sys.argv) < 2:
        print("Usage: python esp_highlight.py [input_file] [output_format]")
        return
        
    input_file = sys.argv[1]
    output_format = sys.argv[2] if len(sys.argv) > 2 else "ansi"
    
    try:
        with open(input_file, 'r') as f:
            content = f.read()
            
        highlighter = ESPHighlighter(output_format)
        highlighted = highlighter.highlight(content)
        
        print(highlighted)
        
    except Exception as e:
        print(f"Error processing file: {e}")

if __name__ == "__main__":
    main()
